---
title: Building a Mobile-Friendly Web UI for My HDHomeRun Tuner
author: Joe
categories: [hdhomerun, docker, nodejs, development, homelab, self-hosted]
tags: [hdhomerun, docker, nodejs, express, htmx, bootstrap, self-hosted, homelab, over-the-air-tv]     # TAG names should always be lowercase
---

## What is hdhomerun-web?

`hdhomerun-web` is a self-hosted, mobile-responsive web app for managing a [HDHomeRun](https://www.silicondust.com/) network tuner — a friendlier alternative to the device's own built-in web interface, which is functional but clearly designed for a desktop browser circa 2010.

You can view the code here: [https://github.com/joescars/hdhomerunweb](https://github.com/joescars/hdhomerunweb)

## Why Did I Build This?

I picked up an HDHomeRun FLEX 4K to pull in over-the-air channels on my home network. The device's built-in web UI works, but it's cramped on a phone, has no TV guide at all, and hides some genuinely useful controls (favoriting channels, hiding ones you don't want) behind a fiddly star-icon interaction. I wanted something I could pull up on my phone from the couch, see what's on, and manage channels without squinting.

## Key Features

- **System Menu** — device info at a glance: friendly name, model, firmware, device ID, tuner count, with a link straight to the device's own system log.
- **TV Guide** — a real program guide (titles, times, episode info, synopses, artwork) per channel, with a "NOW" indicator for what's currently airing.
- **Channel Lineup** — the full channel list, including hidden and unsubscribed channels, with per-channel signal strength/quality and codec info. Filter toggles for Favorites, HD, and Show Hidden, plus one-tap buttons to favorite or hide any channel, synced straight back to the device.
- **Detect Channels** — start or abort a channel scan, with a source selector (Antenna/Cable) and live progress.
- **System Status** — per-tuner status: currently tuned channel, signal strength/quality meters, and network rate, auto-refreshing.
- **Light/dark mode** toggle, remembered across visits.

The whole thing is server-rendered — Node/Express with EJS templates, [htmx](https://htmx.org/) for the small bits of live-updating UI (scan progress, tuner status, per-row favorite/hide toggles), and Bootstrap 5 for styling. No SPA build step, no client-side framework. It ships as a single Docker container configured entirely through a `.env` file.

## The Fun Part: Reverse-Engineering the Undocumented Bits

HDHomeRun's [official HTTP API docs](https://info.hdhomerun.com/info/http_api) cover the basics — `discover.json`, `lineup.json`, streaming URLs — but leave out a bunch of things the device's own web UI clearly uses. Since the device serves its own UI as plain HTML and JavaScript, I could just view-source it to find the rest:

- `lineup.json?show=found` vs `?show=all` — the bare endpoint with no query string actually returns a narrower "favorites-relevant" subset of channels, which is why my first pass showed fewer channels than the device's own lineup page.
- `lineup.post?favorite=<mode><GuideNumber>` — a `+`/`-`/`x` prefix on the channel number sets it to favorite, normal, or hidden. This isn't documented anywhere, but it's exactly what the star icon on the device's own channel list calls.
- `lineup.post?scan=start&source=Antenna` — the scan endpoint accepts an optional source parameter for devices that support multiple tuning sources.

The biggest find, though, was Silicondust's separate [documentation wiki](https://github.com/Silicondust/documentation/wiki), which covers their *cloud* Guide/DVR API (`api.hdhomerun.com`) — full program guide data, search, "up next," and DVR recording rules, all authenticated with a `DeviceAuth` token your own tuner hands out via `discover.json`. As of a recent update, guide data is available with no subscription required. That's what powers the TV Guide page — real program data, titles, episode numbers, synopses, and artwork, with zero guide data baked into the device itself.

## Final Thoughts

This was a satisfying weekend-project kind of build: a small, focused tool that fixes a specific annoyance (checking channels/guide from my phone) using nothing but a device I already own and a wall of undocumented JSON. If you've got an HDHomeRun on your network, the code's up at [github.com/joescars/hdhomerunweb](https://github.com/joescars/hdhomerunweb) — clone it, point `HDHOMERUN_HOST` at your device's IP, and `docker compose up`.
