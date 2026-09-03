---
title: "LunaTV: A Mobile-First HDHomeRun Client for the Browser and Roku"
author: Joe
pin: true
description: >-
  LunaTV is a mobile-first, self-hosted HDHomeRun client for the browser and Roku —
  live TV guide, QSV-accelerated HLS transcoding, and friendlier device admin.
categories: [hdhomerun, docker, nodejs, development, homelab, self-hosted]
tags: [hdhomerun, docker, nodejs, express, htmx, bootstrap, roku, self-hosted, homelab, over-the-air-tv]
---

## What is LunaTV?

[LunaTV](https://joeraio.com/lunatv) is a mobile-first, self-hosted client for watching and browsing live TV from an [HDHomeRun](https://www.silicondust.com/) tuner. It also includes an admin interface that makes device setup and management much friendlier than the tuner’s built-in web UI, plus a sideloadable Roku client powered by the same server.

It runs as a small Docker container, talks directly to the HDHomeRun’s local HTTP API, and gets program data from Silicondust’s cloud Guide API using the `DeviceAuth` token supplied by the tuner.

## Watching Live TV

The default landing page is a TV guide with program titles, times, episode details, synopses, and artwork. A one-tap Watch button opens a full-screen player directly over the guide, so closing it returns you to the same guide position and filters. There is also a denser channel-by-time grid at `/guide/grid` for desktop use.

HDHomeRun tuners stream raw MPEG-2/AC-3, which browsers do not decode natively. LunaTV transcodes an active channel to H.264/AAC HLS with Intel Quick Sync Video (QSV) hardware acceleration, then plays it with [hls.js](https://github.com/video-dev/hls.js) or native HLS on iOS Safari. Playback includes closed captions, and inactive tuner sessions are released automatically.

With an optional, separately-run HDHomeRun RECORD engine, LunaTV can also browse completed recordings, play them through the same HLS pipeline, delete recordings, and schedule the currently playing airing for recording.

## Device Administration

The admin area keeps the HDHomeRun-management features that started the project:

- **System Menu** — device name, model, firmware, device ID, tuner count, and a link to the tuner’s system log.
- **Channel Lineup** — all channels, including hidden and unsubscribed ones, with signal information and controls to favorite or hide a channel on the tuner itself.
- **Detect Channels** — start or abort a scan and select the available source, such as Antenna or Cable.
- **System Status** — auto-refreshing status for every tuner, including tuned channel, signal metrics, and network rate.

## Roku Support

The repository also includes a sideloadable Roku client. It has a live channel-preview option in the guide, recently watched channels, a now/next playback overlay, closed captions, and selectable streaming modes.

The usual H.264 and HEVC modes use the same QSV transcode pipeline as the browser client. An experimental Direct mode remuxes without transcoding and is currently the option that supports ATSC 3.0 (NextGen TV) channels.

## The Stack

LunaTV is server-rendered with Node.js, Express, and EJS—there is no SPA build step. [htmx](https://htmx.org/) handles small live-updating interactions such as scan progress, tuner status, and channel controls, while Bootstrap 5 supplies the admin UI. [jellyfin-ffmpeg](https://github.com/jellyfin/jellyfin-ffmpeg) provides the current Intel media driver support needed for QSV-accelerated streaming.

## Running It

LunaTV starts with `docker compose up -d --build` and listens on port `8080` by default. Configure it in `.env`, using the tuner’s LAN IP address rather than its `.local` mDNS name—the default Docker bridge network cannot resolve mDNS names. Live playback requires an Intel GPU with QSV support exposed to the container through `/dev/dri`; guide, administration, and other non-playback features work without it.

The project is available at [joeraio.com/lunatv](https://joeraio.com/lunatv). If you have an HDHomeRun on your network, clone it, point `HDHOMERUN_HOST` at the tuner’s LAN IP, and run `docker compose up -d --build`.
