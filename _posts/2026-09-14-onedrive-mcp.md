---
title: "A Read-Only MCP Server for OneDrive, Because I Wanted My Agents to Actually See My Files"
description: Building a read-only Model Context Protocol server for personal OneDrive via Microsoft Graph, with Hermes, VS Code/GitHub Copilot, and Open WebUI setup.
excerpt: My agents can browse the web, write code, and manage my calendar — but they couldn't look at a single file in my own OneDrive. Here's the small, deliberately read-only MCP server that fixes that.
author: Joe
categories: [ai, mcp, microsoft-graph, self-hosted, development]
tags: [mcp, onedrive, microsoft-graph, hermes, openwebui, msal, python, self-hosted]
---

I run most of my day-to-day AI work through [Hermes](https://github.com/NousResearch/hermes) as my primary agent interface, with [Open WebUI](https://github.com/open-webui/open-webui) in the mix for a few other things. Both are good at reaching out to the world — web search, code execution, calendar, email. What neither of them could do was look inside my own OneDrive. If I wanted an agent to find a document, check a file's metadata, or pull something down to work on locally, I had to go get it myself first and paste it in.

That's a solved problem in principle — Model Context Protocol (MCP) exists exactly to let an agent reach out to a new capability through a small, well-defined server. So I built one for OneDrive.

*Updated September 15, 2026, to reflect the [current implementation](https://github.com/joescars/onedrive-mcp/tree/84ccf59d1e8643a0386bde840256333392312752), including the reliability and security hardening, VS Code setup, and updated dependency locks.*

## Scoping it down on purpose

The first decision mattered more than any line of code: this server is **read-only toward OneDrive**. It can search, list, and download. It exposes no tool to upload, delete, rename, or move anything in the drive. I didn't want to reason about "what happens if the agent decides to reorganize my files" — I wanted those operations left out of the tool surface, not just discouraged by a system prompt. Downloading does write a copy to local disk; it never modifies the OneDrive source.

That constraint shows up in both the requested permissions and the current implementation. Authentication requests only delegated read scopes:

```python
SCOPES = ["Files.Read", "Files.Read.All"]
```

The Graph client's `_get()` helper hardcodes `requests.get`. Downloads can also use a separate `requests.get` to stream from Graph's pre-authenticated download URL. Both paths are GET-only. The assertion inside `_get()` documents its intent; it isn't a global safeguard that would catch someone adding a write call elsewhere in a future edit. The read-only design rests on the scopes, the exposed tools, and keeping those network paths read-only.

Read-only doesn't mean private to the server, either. Names, paths, and metadata returned by tools enter the client's chat context and are subject to that client's and model provider's data policies. The server deliberately leaves pre-authenticated download URLs out of metadata responses and sanitizes transfer errors so those file-access links don't leak into transcripts.

## The auth flow: device code, no browser needed

This runs on a headless Linux box, and it's my *personal* OneDrive — not a work tenant. That combination pointed at a specific setup:

- **MSAL's OAuth2 device code flow.** No redirect URI, no browser needed on the server itself. You run a one-time setup script, it prints a short code and a URL (`microsoft.com/devicelogin`), and you complete the sign-in on literally any device with a browser — your phone works fine.
- **A personal-account app registration** in Microsoft's identity platform. This isn't deploying anything to paid Azure compute: the server runs on my own box. Register an app for "Personal Microsoft accounts only," add delegated `Files.Read` and `Files.Read.All` permissions, enable "Allow public client flows," and copy the client ID. No client secret is needed. The server defaults to the `consumers` tenant; this setup is not for OneDrive for Business or SharePoint.
- **A persisted, refresh-capable token cache**, owner-only (`0600`) on Linux. The server silently refreshes access tokens, including a forced refresh after a rejected token. It's normally a one-time sign-in, not a forever guarantee: revoked or expired refresh credentials mean running setup again. Cache updates now use a cross-process lock and atomic replacement so multiple MCP processes sharing the cache don't overwrite each other's updates.

Linux is the supported production platform. The token cache is plaintext protected by filesystem permissions, not encrypted. Native Windows/non-POSIX use is development-only: `chmod` doesn't establish owner-only Windows ACLs. For a Windows desktop, running the server on Linux through WSL or Remote-SSH is the documented route; in WSL, keep the checkout, cache, and downloads in the Linux home filesystem.

## Five tools, that's it

The MCP server exposes exactly this surface:

- `search_onedrive(query, top=20, next_link=None)` — Graph full-text search across file/folder names and content, one page at a time
- `list_folder(path='/', top=50, next_link=None)` — list a folder's contents, one page at a time
- `get_item_metadata(path_or_id)` — normalized metadata for one file or folder, without a pre-authenticated download URL
- `download_file(path_or_id, dest_filename=None)` — pull a file into `DOWNLOAD_DIR`, returning its local path and metadata, not its contents
- `get_drive_info()` — quota and owner info, mostly useful as a smoke test

Folder listing, metadata lookup, and downloads accept human-readable paths (`/Documents/report.pdf`) or raw Graph item IDs. Search takes a query, and drive info takes no arguments. Responses use normalized JSON shapes rather than raw Graph API payloads.

Search and listing now return `next_link` and `has_more`; `top` must be an integer from 1 to 200. Search also returns `query`, `count`, and `items`, with `count` referring to the current page, not the whole result set. To keep paging, pass `next_link` back to the same tool with the same query or folder and the same `top`. Despite the name, it's an opaque, signed continuation token — not a URL to fetch yourself — and expires after one hour or a server restart.

Downloads have some deliberately boring guardrails now, too. Defaults are **100 MiB per file** and **1 GiB for the download directory**, configurable with `MAX_DOWNLOAD_BYTES` and `MAX_DOWNLOAD_DIR_BYTES`. Transfers sharing that directory are serialized to enforce the budget, with limits checked before and during streaming. A file is published only after transfer and size validation succeed; failed partial transfers are cleaned up, and existing destinations are never overwritten. `dest_filename` must be a bare filename, not an arbitrary path. The atomic publication step needs a filesystem with hard-link support, such as ext4.

One important distinction: "local" means **the machine running the MCP server**. If that's a remote Linux host or WSL, downloading doesn't put the file on the desktop displaying chat. The client needs separate filesystem access to read or summarize the downloaded contents.

## The bug that actually taught me something

Everything worked in local testing. Then I registered it with Hermes and got: `No signed-in account found in the token cache` — even though the one-time sign-in had clearly succeeded and the token cache file existed right where I expected it.

The cause was a classic one: the default token cache path was relative, `./token_cache.bin`, so it depended on the *current working directory of whatever process launched the script*. In my Hermes setup, that was its install directory, not the project's folder. `auth.py` also used implicit `.env` discovery through `load_dotenv()` with no arguments. That discovery is context-dependent, not simply always cwd-based, but neither setting should depend on how an MCP host launches the process. The server wasn't reliably finding its project-local configuration and cache.

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

## Three front doors: Hermes, VS Code, and Open WebUI

This server uses MCP over stdio. Hermes and VS Code can connect directly; Open WebUI's OpenAPI Tool Server route needs an HTTP bridge:

- **Hermes** connects directly over stdio. Configure an `onedrive` entry under `mcpServers`, with `command` pointing to the project's virtual-environment Python and `args` containing the absolute path to `server.py`. The client starts the process; you don't need a separate running server.
- **VS Code / GitHub Copilot** also connects over stdio, with no `mcpo`, HTTP port, or bridge API key. Its MCP configuration uses `servers`, not `mcpServers`, with `"type": "stdio"` and the same interpreter/script paths. For Linux through WSL or Remote-SSH, use **MCP: Open Remote User Configuration** in that remote window. Start it through **MCP: List Servers**, approve trust when prompted, and enable the tools in Agent chat. Complete device-code sign-in on the server host first; chat doesn't perform that setup for you.
- **Open WebUI** goes through [`mcpo`](https://github.com/open-webui/mcpo), which wraps the stdio server as an OpenAPI HTTP server. Use the included `scripts/run_mcp_bridge.py` launcher: it defaults to `127.0.0.1` and requires `MCPO_API_KEY`, protecting the API and documentation endpoints. Load the key from an owner-only `0600` env file rather than passing it in command-line arguments, and enter the same key in Open WebUI's Tool Server settings.

Localhost binding alone isn't authentication — other accounts on the same machine can reach a loopback port. Keep the API key even behind an authenticated reverse proxy, and don't expose the bridge directly to the internet. The [README](https://github.com/joescars/onedrive-mcp#6-expose-to-open-webui-via-mcpo) has the key-generation steps and a systemd template with writable runtime state separated from the read-only source and interpreter.

## Get it

The whole thing — server, MSAL auth, mocked Graph/MSAL tests, stdio integration checks, a sign-in setup script, and a fairly thorough README covering app registration and client setup — is public:

**[github.com/joescars/onedrive-mcp](https://github.com/joescars/onedrive-mcp)**

On Linux with Python 3.10+ (3.12 recommended; the optional `mcpo` bridge needs 3.11+), after completing the app registration:

```bash
git clone https://github.com/joescars/onedrive-mcp
cd onedrive-mcp
python3 -m venv venv
./venv/bin/pip install --require-hashes -r requirements.lock
cp .env.example .env   # fill in AZURE_CLIENT_ID
./venv/bin/python scripts/setup_auth.py
```

For Open WebUI, also install `requirements-bridge.lock` with `--require-hashes` into that same virtual environment, then follow the bridge setup above. The project now ships hash-locked dependencies and CI checks, including scheduled dependency audits. Use the current lockfiles rather than bypassing the hashes if your package index reports an unavailable version; the README covers that troubleshooting case.

If you're already running Hermes, VS Code/GitHub Copilot, or Open WebUI and want your agents to find your OneDrive files instead of you copy-pasting them in, that's the idea: a small read-only tool surface, with a little more care around what gets cached, downloaded, and exposed to the client.
