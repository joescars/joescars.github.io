---
title: "A Microsoft Foundry MAI Image Plugin for Hermes Agent"
description: A reusable Hermes Agent image-generation plugin for Microsoft Foundry MAI models, with text-to-image generation, image editing, and Web UI configuration.
excerpt: Microsoft Foundry's MAI image endpoint is powerful but not OpenAI-shaped. Here's a reusable Hermes plugin that speaks its native API and works from the CLI or Web UI.
author: Joe
categories: [ai, azure, hermes, development]
tags: [hermes, azure, microsoft-foundry, mai-image, image-generation, python, open-source, plugins]
---

I've been using [Hermes Agent](https://github.com/NousResearch/hermes-agent) as my main agent interface, and I wanted image generation to fit into the same workflow as everything else: one tool call from the agent, a local cached result, and the option to use it from either the terminal or Hermes Web UI.

Microsoft Foundry's MAI image models made a good target. The models are available through Azure, but their image API is not quite the same as the standard OpenAI image endpoint. That difference is small at the HTTP level and large enough to make a generic integration fail.

So I built a standalone plugin:

**[github.com/joescars/hermes-mai-image](https://github.com/joescars/hermes-mai-image)**

## Why a separate plugin?

Hermes already has an image-generation provider interface. A provider plugin can register an image backend without modifying Hermes core, which keeps the MAI-specific behavior isolated and gives other people a straightforward installation path.

The plugin uses the normal Hermes provider lifecycle:

- It registers as `azure-mai`.
- It declares its setup fields to Hermes.
- It exposes a model catalog and a default model.
- It advertises text-to-image and image-to-image capabilities.
- It returns the standard Hermes image response.

That last point matters. Hermes can route the result to its existing image-generation flow instead of needing a MAI-specific tool everywhere a generated image might be used.

## The MAI API difference

MAI image generation is served from a Microsoft-managed path beneath the Foundry resource endpoint:

```text
https://<resource>.services.ai.azure.com/mai/v1/images/generations
```

The plugin sends the API key in the `api-key` header. A generation request looks like this:

```json
{
  "model": "your-deployment-name",
  "prompt": "A red fox in a snowy forest at sunrise",
  "width": 1536,
  "height": 1024
}
```

The endpoint is intentionally configured as the resource root:

```text
https://your-resource.services.ai.azure.com
```

You do not add `/mai/v1` to the configured endpoint. The plugin appends the MAI path itself, which avoids duplicated paths and makes the same configuration work for both generation and editing.

## Generation and editing

The first version supports both sides of the MAI image API.

Text-to-image requests go to:

```text
POST /mai/v1/images/generations
```

Image edits go to:

```text
POST /mai/v1/images/edits
```

For an edit, the plugin sends the source PNG or JPEG as multipart form data along with the deployment name and prompt. The source can be a local file path or an HTTP(S) URL through Hermes' image-generation interface.

The output is normalized into Hermes' image cache under:

```text
$HERMES_HOME/cache/images/
```

That gives the result a stable local path instead of relying on a short-lived service URL.

## Install it from GitHub

The package is currently distributed from GitHub:

```bash
/usr/local/lib/hermes-agent/venv/bin/pip install --upgrade \\
  "git+https://github.com/joescars/hermes-mai-image.git"
```

Then enable the plugin and Hermes image-generation toolset:

```bash
hermes plugins enable azure-mai-image
hermes tools enable image_gen
```

Plugins load at process startup, so restart long-running Hermes processes after installation:

```bash
hermes gateway restart
hermes dashboard --stop
hermes dashboard
```

No operating-system reboot is required.

## Configure it from Hermes

From the CLI, run:

```bash
hermes tools
```

Select:

```text
Image Generation → Microsoft Foundry MAI
```

The provider asks for:

- `MAI_FOUNDRY_API_KEY`
- `MAI_FOUNDRY_ENDPOINT`

The endpoint is the resource root, not the full `/mai/v1/images/generations` URL. The deployment name can be selected through the model picker or set explicitly as:

```dotenv
MAI_IMAGE_MODEL=your-deployment-name
```

The Hermes Web UI uses the same provider setup schema. Open **Tools → Image Generation**, select **Microsoft Foundry MAI**, enter the key and endpoint, save, and choose the deployment if the model picker is shown.

Credentials are stored in the active Hermes profile's `.env`. The plugin never requires putting API keys in `config.yaml` or the repository.

## A small real-world integration lesson

The hardest part was not the HTTP request. It was making the plugin appear consistently across Hermes surfaces.

Hermes discovers pip-installed providers through the `hermes_agent.plugins` entry-point group. The entry-point target needs to match Hermes' discovery contract: a bare module name for a provider module that registers itself on import. Using a callable-style target looked reasonable, but Hermes treated it as a general plugin and reported that the plugin had no `register()` function.

The fix was one line in `pyproject.toml`:

```toml
[project.entry-points."hermes_agent.plugins"]
azure-mai-image = "hermes_mai_image"
```

This is a useful reminder that plugin systems often have two related but distinct contracts: the package metadata that discovers a plugin and the runtime hook that registers the provider. Both need to be tested from a fresh process.

## Testing without an Azure key

The repository includes mocked tests for the important request contracts:

- `/models` discovery parsing
- Generation endpoint and `api-key` authentication
- Generation dimensions
- Multipart image editing
- Missing-credential errors

Run them with:

```bash
python3 -m venv .venv
. .venv/bin/activate
pip install -e '.[test]'
pytest -q
```

The tests do not make live Azure requests and do not require an API key. The package also builds as a wheel, so the same source can move from GitHub installation to PyPI later.

## What happens when Hermes updates?

A normal Hermes update does not normally delete the separately installed plugin or the profile configuration. After an update, check:

```bash
hermes plugins list
```

If the plugin remains enabled, restart Hermes processes. If an update replaces the Hermes Python virtual environment, reinstall the plugin into the new environment:

```bash
/usr/local/lib/hermes-agent/venv/bin/pip install --upgrade \\
  "git+https://github.com/joescars/hermes-mai-image.git"
```

The API key and endpoint normally remain in the Hermes profile, so they do not need to be entered again unless the profile or its `.env` file was reset.

## Try it

After configuring a deployment, a simple test is:

```bash
hermes chat -q "Generate an image of a red fox sitting in a snowy forest at sunrise."
```

For an edit:

```bash
hermes chat -q "Edit /absolute/path/to/source.png: turn the scene into a nighttime scene with moonlight."
```

The goal is not to make MAI pretend to be OpenAI. It is to let Hermes speak MAI's native API while presenting the same provider experience users already understand.

If you're using Microsoft Foundry MAI and Hermes, the plugin is available here:

**[github.com/joescars/hermes-mai-image](https://github.com/joescars/hermes-mai-image)**
