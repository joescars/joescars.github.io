---
title: "A Read-Only OneDrive MCP Server for AI Agents"
description: Search, browse, and download personal OneDrive files from Hermes, Open WebUI, or VS Code. Setup, Microsoft Graph permissions, examples, and privacy limits.
excerpt: I built a small MCP server so my agents could find files in my personal OneDrive. Here's how to connect it, what it can actually do, and where read-only access still needs care.
author: Joe
categories: [ai, mcp, microsoft-graph, self-hosted, development]
tags: [mcp, onedrive, microsoft-graph, hermes, openwebui, msal, python, self-hosted]
---

I run most of my day-to-day AI work through [Hermes](https://github.com/NousResearch/hermes-agent) as my primary agent interface, with [Open WebUI](https://github.com/open-webui/open-webui) in the mix for a few other things. My setup could reach out to the web and run code, but it had no connection to my personal OneDrive. If I wanted an agent to find a document, check a file's metadata, or pull something down to work on locally, I had to go get it myself first and paste it in.

That's a solved problem in principle — Model Context Protocol (MCP) exists exactly to let an agent reach out to a new capability through a small, well-defined server. So I built one for OneDrive.

**[Get the OneDrive MCP server on GitHub](https://github.com/joescars/onedrive-mcp).** It targets **personal Microsoft accounts on Linux**, with Python 3.10 or newer. It is not a OneDrive for Business or SharePoint connector, a sync client, or a document-parsing service. The optional Open WebUI bridge needs Python 3.11 or newer.

## Scoping it down on purpose

The first decision mattered more than any line of code: this server is **read-only with respect to OneDrive**. It can search, list, inspect metadata, and download. It exposes no tools to upload, delete, rename, or move cloud files. I wanted the agent's available tools to reflect that constraint, rather than relying on a system prompt to discourage changes.

The Graph operations use a GET-only helper. That is an implementation constraint, not a guarantee that future edits cannot introduce writes. In particular, assigning `requests.get` to a variable and asserting that it is still `requests.get` does not enforce a security boundary. The exposed tool surface, read-only delegated permissions, and regression tests matter more than that assertion.

Read-only also does **not** mean risk-free. Downloads write to local disk, and file names, paths, and metadata enter the client's context. If another tool reads a downloaded document, its contents may be sent to the model provider. Use an account whose files you intend to expose, protect the token cache, and treat instructions inside retrieved documents as untrusted content.

## The auth flow: no browser on the server

This runs on a headless Linux box, and it's my *personal* OneDrive — not a work tenant. That combination pointed at a specific setup:

- **MSAL's OAuth 2.0 device code flow.** No client secret or redirect URI, and no browser needed on the server itself. The setup script prints a code and a sign-in URL; you complete sign-in on a device with a browser. Only approve a device code you initiated yourself.
- **An app registration that accepts personal Microsoft accounts.** Enable **Allow public client flows**, add the Microsoft Graph **delegated** permissions `Files.Read` and `Files.Read.All`, and use the Application (client) ID in `.env`. Keep `AZURE_TENANT_ID=consumers` for personal-account sign-in. These scopes allow reads, but are not a one-folder access boundary.
- **A persisted, refresh-capable token cache**, protected with mode `0600` on Linux. The server normally refreshes access tokens silently, but revoked or expired credentials can require another interactive sign-in. The cache is plaintext credential material, not an encrypted vault.

The registration prerequisite is easy to overlook: a personal OneDrive account is not itself an Entra directory where you can necessarily register an app. Microsoft's [current registration guide](https://learn.microsoft.com/en-us/entra/identity-platform/quickstart-register-app#prerequisites) lists an Azure account with an active subscription, a tenant, and registration permissions. This server runs locally and does not deploy Azure compute, but I would not promise that every reader can complete registration without additional account setup. Follow the repository's [setup guide](https://github.com/joescars/onedrive-mcp/blob/main/docs/setup.md) before installing.

## Five tools, that's it

The MCP server exposes exactly this surface:

| Tool                                             | What it returns                                                     |
| ------------------------------------------------ | ------------------------------------------------------------------- |
| `search_onedrive(query, top=20, next_link=None)` | One page of matches from Graph's search index                       |
| `list_folder(path="/", top=50, next_link=None)`  | One page of a folder's children                                     |
| `get_item_metadata(path_or_id)`                  | Selected metadata, including size, modification time, and MIME type |
| `download_file(path_or_id, dest_filename=None)`  | A local file path, byte count, MIME type, and source name           |
| `get_drive_info()`                               | Drive, owner, and quota information                                 |

Folder listings, metadata lookups, and downloads accept a leading-slash path such as `/Documents/report.pdf` or a raw Graph item ID. Search takes text; drive info takes no arguments. Responses are normalized JSON, not raw Graph payloads or document contents.

Search and listings are **paginated**, not exhaustive in one call. `top` is a page size from 1 to 200. When `has_more` is true, pass the returned opaque `next_link` to the same tool with the original query or path and `top`. Continue until `has_more` is false. Continuations expire after an hour or a server restart; restart the search or listing in that case.

Search relies on Microsoft's index, not a scan of every file's bytes. An empty search result is not proof that a file does not exist. Browse a known folder or look up an exact path when you need to check a particular document.

## A useful first workflow

After setup, start with discovery before granting a download:

1. "Use OneDrive to list `/Documents` without downloading anything. Tell me whether there are more pages."
2. "Search OneDrive for `invoice`. Follow all result pages and show the names, paths, and modification dates."
3. "Get metadata for `/Documents/report.pdf`. Do not download it yet."
4. "Download `/Documents/report.pdf` as `report-copy.pdf`."

Use a path that actually exists in your drive. Downloads stay under `DOWNLOAD_DIR` **on the machine running the MCP server**, which might be a remote Linux host rather than your laptop. Existing files are not overwritten. Defaults are 100 MiB per file and 1 GiB for the download directory; both are configurable.

This server does not extract PDF or Office text, OCR images, build a vector index, or automatically attach downloads to Open WebUI. Summarizing a downloaded document needs a separate file-reading or ingestion capability with access to that host's file. That distinction matters: finding a document and reading its contents are separate steps.

## The bug that actually taught me something

Everything worked in local testing. Then I registered it with Hermes and got: `No signed-in account found in the token cache` — even though the one-time sign-in had clearly succeeded and the token cache file existed right where I expected it.

The important trap was the relative token-cache path, `./token_cache.bin`. A bare relative file path resolves against the launching process's working directory, which an MCP host need not set to the project's folder. The server could therefore look for a different cache from the one the setup script had written.

There is a related but distinct detail: `load_dotenv()` without an explicit path uses discovery rules. In ordinary script execution it searches upward from the caller's file; interactive or debugger contexts can use the working directory. It is not accurate to say it always searches from cwd. Explicit paths remove that ambiguity as well.

The fix was straightforward once diagnosed: resolve both the `.env` load and any relative path default against the script's own file location, not the process's cwd:

```python
import os
from pathlib import Path

from dotenv import load_dotenv

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

## Connect Hermes, Open WebUI, or VS Code

MCP supports more than one transport; **this server implements stdio**. The client launches the process, so use absolute paths to both its virtual environment and `server.py`. You do not need to leave a separate stdio server running.

For **Hermes**, merge this into `~/.hermes/config.yaml`, replacing the example paths:

```yaml
mcp_servers:
    onedrive:
        command: "/home/you/onedrive-mcp/venv/bin/python"
        args: ["/home/you/onedrive-mcp/server.py"]
```

Start a new session and ask for `get_drive_info`. The repository's [client guide](https://github.com/joescars/onedrive-mcp/blob/main/docs/clients.md) also covers **VS Code with GitHub Copilot**, including WSL and Remote-SSH. VS Code uses a JSON `servers` configuration, not Hermes's YAML `mcp_servers`.

For **Open WebUI**, follow the [authenticated bridge guide](https://github.com/joescars/onedrive-mcp/blob/main/docs/deployment.md). [`mcpo`](https://github.com/open-webui/mcpo) converts this stdio server to an OpenAPI HTTP tool server. The included launcher requires an API key even on loopback. Keep it private; do not expose an unauthenticated endpoint. If Open WebUI runs in Docker, its `127.0.0.1` is the container, not the bridge host. Native MCP HTTP support in a client does not make a stdio process or an OpenAPI bridge a native MCP HTTP endpoint.

## Install and verify

After completing the app registration, install on your Linux host:

**[github.com/joescars/onedrive-mcp](https://github.com/joescars/onedrive-mcp)**

```bash
git clone https://github.com/joescars/onedrive-mcp.git
cd onedrive-mcp
python3 -m venv venv
./venv/bin/pip install --require-hashes -r requirements.lock
cp .env.example .env
chmod 600 .env
```

Edit `.env` and replace the placeholder `AZURE_CLIENT_ID` with your Application (client) ID. Then sign in and verify access:

```bash
./venv/bin/python scripts/setup_auth.py
./venv/bin/python scripts/smoke_test.py
```

The live smoke test reads drive information and the root folder, without downloading or modifying files. Run it as the same OS user on the same host that will run the MCP server. The separate `./venv/bin/python -m pytest -q` suite uses mocked Graph/MSAL calls and does not need account credentials.

If setup fails, these are the first checks I would make:

| Symptom                                         | First check                                                                                             |
| ----------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| No signed-in account or refresh failure         | Run setup auth on the server host as its OS user; confirm `TOKEN_CACHE_PATH` and restart the MCP server |
| App not found or client secret requested        | Check the client ID, supported personal-account type, and **Allow public client flows**                 |
| Graph returns 403                               | Check delegated permissions, consent, and access to that item                                           |
| Download succeeds but the client cannot read it | The returned path is on the server host; configure a separate reader or ingestion step                  |

The repository has more [troubleshooting steps](https://github.com/joescars/onedrive-mcp/blob/main/docs/troubleshooting.md), a [tool reference](https://github.com/joescars/onedrive-mcp/blob/main/docs/tools.md), and the [security model](https://github.com/joescars/onedrive-mcp/blob/main/docs/security.md). The useful outcome is deliberately small: an agent can find the right file, show me what it found, and download an approved copy without a tool that can reorganize my OneDrive.
