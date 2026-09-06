---
title: "Bridging Open WebUI and Azure's MAI Image Models with a Tiny OpenAI-Compatible Shim"
description: Why Open WebUI can't talk to Azure AI Foundry's MAI-Image models directly, and the small proxy that fixes it.
excerpt: A custom Azure image endpoint, an OpenAI-shaped client, and the 404s in between — here's the shim that closes the gap.
author: Joe
categories: [azure, openwebui, self-hosted, development]
tags: [azure, openai, openwebui, mai-image, self-hosted, docker, nodejs, image generation]
---

I've been running a small internal admin site ([raiotribe-web](https://github.com/joescars/raiotribe-web)) with an image generator page that calls Azure AI Foundry's MAI-Image models directly. It works great from the browser. Naturally, the next thing I wanted was to point [Open WebUI](https://github.com/open-webui/open-webui) at the same model, since I already have it running against other models in the same subscription. That's where things got annoying.

## The problem

Open WebUI's built-in "OpenAI" image generation provider assumes the standard OpenAI (and standard Azure OpenAI DALL-E) image API shape:

- Auth via `Authorization: Bearer <key>`
- A request body like `{ model, prompt, size: "1024x1024", n }`
- For anything hosted on an `azure.com` domain, it tries to be helpful and rewrites the request into the classic Azure OpenAI deployment path: `/openai/deployments/{model}/images/generations?api-version=...`

MAI-Image models on Azure AI Foundry don't live at that path. Mine is served from something like `https://<resource>.services.ai.azure.com/mai/v1/images/generations`, and it expects:

- Auth via an `api-key` header, not Bearer
- A body shaped like `{ model, prompt, width, height }` — no `size` string at all

Point Open WebUI straight at that endpoint and you get hit twice at once: the auth scheme is wrong, and Open WebUI's Azure auto-detection tries to rewrite the URL into a deployment path that doesn't exist on this resource. What comes back isn't a helpful "wrong auth" error — it's a flat `400: Resource Not Found`, because Azure is telling you, accurately, that the path it built doesn't map to anything.

There's no setting in Open WebUI's UI to fix either half of this. The auth header and the URL-building logic are hardcoded to the standard shape.

## The fix: a tiny translation layer

Rather than trying to coax Open WebUI into behaving, I wrote a small shim that sits in between:

- To Open WebUI, it looks like a normal OpenAI-compatible image endpoint — Bearer auth, `size: "WxH"` bodies.
- To Azure, it speaks the MAI-Image dialect — `api-key` header, `width`/`height` integers.

It's a translation layer, not a reimplementation: prompt and model pass straight through, `size` gets split into `width`/`height`, and the auth header gets swapped before the request goes upstream. The response comes back already in OpenAI's shape (`{ data: [{ b64_json }] }`), so nothing needs to change on the way out either.

## Two more gotchas along the way

Getting the translation right wasn't quite the end of it. Two more issues showed up only once I started testing against the real client:

**A trailing space in the Base URL field.** Open WebUI concatenates whatever's in the Base URL box with the request path. A stray trailing space in that field gets URL-encoded straight into the path (`.../shim%20/images/generations`), which 404s in a way that looks nothing like "you typed a space." I only caught this by adding request logging and looking at the literal path Open WebUI was requesting.

**Open WebUI always appends `/v1/images/generations`.** Regardless of whether your Base URL already ends in `/v1`, Open WebUI tacks the full `/v1/images/generations` suffix onto it. So the shim answers at both `/images/generations` and `/v1/images/generations` — whichever convention the client assumes, it works.

Both of these were invisible until I could see the exact request hitting the server. That's worth remembering any time an integration "just 404s" for no obvious reason: log the actual path and headers before guessing.

## Using it

The shim is a standalone, dependency-free Node service — no Express, no npm packages, just `node:http` — so it's a small Docker image with nothing to audit:

**[github.com/joescars/openwebui-azure-image-shim](https://github.com/joescars/openwebui-azure-image-shim)**

```bash
cp .env.example .env
# fill in AZURE_API_KEY and AZURE_IMAGE_ENDPOINT
docker compose up --build -d
```

Then in Open WebUI's Admin Panel → Settings → Images, point Base URL at wherever the shim is running, set the API key to the same `AZURE_API_KEY`, and pick a model name from your `ALLOWED_MODELS` list. Everything else — endpoint URL, model allowlist, size limits — is configurable via environment variables, so it isn't tied to MAI-Image specifically; it should work for any Azure AI Foundry image endpoint that follows this same non-standard shape.

## Final thoughts

None of this is exotic — it's a couple dozen lines of request translation. But "the client and the server both claim to speak OpenAI's API, and yet nothing works" is a specific enough kind of frustrating that I suspect I'm not the only one who's hit it with Azure AI Foundry's newer image models. If you're trying to wire Open WebUI (or anything else that only knows the standard OpenAI image shape) up to one of these endpoints, hopefully this saves you the hour of staring at "Resource Not Found" that it cost me.
