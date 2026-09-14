---
title: "A Read-Only MCP Server for OneDrive, Because I Wanted My Agents to Actually See My Files"
description: Building a Model Context Protocol server that lets Hermes and Open WebUI search and download from personal OneDrive via Microsoft Graph — read-only, by design.
excerpt: My agents can browse the web, write code, and manage my calendar — but they couldn't look at a single file in my own OneDrive. Here's the small, deliberately read-only MCP server that fixes that.
author: Joe
categories: [ai, mcp, microsoft-graph, self-hosted, development]
tags: [mcp, onedrive, microsoft-graph, hermes, openwebui, msal, python, self-hosted]
---

I run most of my day-to-day AI work through [Hermes](https://github.com/NousResearch/hermes) as my primary agent interface, with [Open WebUI](https://github.com/open-webui/open-webui) in the mix for a few other things. Both are good at reaching out to the world — web search, code execution, calendar, email. What neither of them could do was look inside my own OneDrive. If I wanted an agent to find a document, check a file's metadata, or pull something down to work on locally, I had to go get it myself first and paste it in.

That's a solved problem in principle — Model Context Protocol (MCP) exists exactly to let an agent reach out to a new capability through a small, well-defined server. So I built one for OneDrive.

## Scoping it down on purpose

The first decision mattered more than any line of code: this server is **read-only**, full stop. It can search, list, and download. It cannot upload, delete, rename, or move anything. I didn't want to reason about "what happens if the agent decides to reorganize my files" — I wanted that entire class of mistake to be structurally impossible, not just discouraged by a system prompt.

That constraint shows up directly in the code, not just the docs. The Graph API client has exactly one function that's allowed to touch the network, it hardcodes `requests.get`, and it asserts on that fact:

```python
def _get(url, *, params=None, stream=False, ...):
    """The ONLY function in this codebase allowed to call the network for
    Graph. It always performs requests.get (READ ONLY — see module docstring)."""
    http_verb = requests.get
    assert http_verb is requests.get, "graph_client must only ever perform GET requests"
    ...
```

If someone (including a future me, or an agent "helpfully" extending the code) ever tries to sneak in a `POST` or `DELETE`, that assertion is there to make the mistake loud instead of silent.

## The auth flow: device code, no browser needed

This runs on a headless Linux box, and it's my *personal* OneDrive — not a work tenant. That combination pointed at a specific setup:

- **MSAL's OAuth2 device code flow.** No redirect URI, no browser needed on the server itself. You run a one-time setup script, it prints a short code and a URL (`microsoft.com/devicelogin`), and you complete the sign-in on literally any device with a browser — your phone works fine.
- **A personal-account app registration** in Azure's free App Registration flow. Worth saying clearly: this doesn't require an Azure subscription, billing, or any cloud spend. "Azure Portal" here is just Microsoft's identity platform UI — you register an app, mark it "Personal Microsoft accounts only," and copy a client ID. Two minutes, no cost.
- **A persisted, refresh-capable token cache**, chmod 600, so the one-time sign-in is actually one-time. The server silently refreshes access tokens on its own after that.

## Five tools, that's it

The MCP server exposes exactly this surface:

- `search_onedrive(query, top)` — full-text search across the drive
- `list_folder(path, top)` — list a folder's contents
- `get_item_metadata(path_or_id)` — full metadata for one file or folder
- `download_file(path_or_id, dest_filename)` — pull a file to local disk, return the path
- `get_drive_info()` — quota and owner info, mostly useful as a smoke test

Every tool accepts either a human-readable path (`/Documents/report.pdf`) or a raw Graph item ID, and every response comes back in a clean, stable JSON shape rather than raw Graph API payloads.

## The bug that actually taught me something

Everything worked in local testing. Then I registered it with Hermes and got: `No signed-in account found in the token cache` — even though the one-time sign-in had clearly succeeded and the token cache file existed right where I expected it.

The cause was a classic one: `auth.py` called `load_dotenv()` with no arguments, and the default token cache path was a relative one, `./token_cache.bin`. Both resolve relative to the *current working directory of whatever process launches the script* — and Hermes doesn't launch MCP servers from the project's own folder. It launches them from its own install directory. So the server was silently looking for `.env` and the token cache in the wrong place entirely, finding neither, and falling back to "no account signed in."

The fix was straightforward once diagnosed: resolve both the `.env` load and any relative path default against the script's own file location, not the process's cwd:

```python
PROJECT_ROOT = Path(__file__).resolve().parent
load_dotenv(PROJECT_ROOT / ".env")

def get_token_cache_path() -> Path:
    raw = os.environ.get("TOKEN_CACHE_PATH", "./token_cache.bin")
    path = Path(raw).expanduser()
    if not path.is_absolute():
        path = PROJECT_ROOT / path
    return path.resolve()
```

It's a small fix, but it's the kind of bug that's specific to how MCP servers get launched — worth remembering any time a script "works standalone but not from the host," because the host's working directory is rarely the same as the script's own folder.

## Two front doors: Hermes and Open WebUI

Since MCP is stdio-native but Open WebUI's Tool Server feature speaks OpenAPI/HTTP, this server needed two ways in:

- **Hermes** connects directly over stdio — `hermes mcp add onedrive --command /path/to/venv/bin/python --args /path/to/server.py`, and it's live in the next session.
- **Open WebUI** goes through [`mcpo`](https://github.com/open-webui/mcpo), which wraps any stdio MCP server as an OpenAPI HTTP server. Bind it to localhost (or put it behind an authenticated reverse proxy) and add it as a Tool Server in Open WebUI's settings.

## Get it

The whole thing — server, MSAL auth, unit tests (mocked, no real network calls), a one-time setup script, and a fairly thorough README covering the Azure app registration steps — is public:

**[github.com/joescars/onedrive-mcp](https://github.com/joescars/onedrive-mcp)**

```bash
git clone https://github.com/joescars/onedrive-mcp
cd onedrive-mcp
python3 -m venv venv && ./venv/bin/pip install -r requirements.txt
cp .env.example .env   # fill in AZURE_CLIENT_ID
./venv/bin/python scripts/setup_auth.py
```

If you're already running Hermes or Open WebUI and want your agents to actually see your files instead of you copy-pasting them in, it should take about ten minutes end to end — most of that is the free Azure app registration, not the code.
