---
name: vidaa-streaming
description: Operate authorized Hisense/VIDAA TVs and stream local media to the TV browser. Use for TV discovery, VIDAA pairing, status/app control, LAN video playback, codec/network diagnostics, and playback verification.
---

# VIDAA Streaming

Use this skill only with a TV the user owns or is authorized to operate.

This package is a reusable workflow reference. It expects an existing VIDAA
project containing `tools/tvctl.py`, `tools/devserver.py`, `app/player.html`,
and a Python virtual environment. Set `VIDAA_PROJECT_ROOT` when the current
working directory is not that project.

## Capabilities

- VIDAA MQTT status, device info, app discovery, remote keys, volume, source, and app launch.
- Local HTTP video delivery from the Mac to the TV browser.
- Browser-player playback verification using HTTP Range responses and TV beacons.
- Diagnosis of LAN routing, browser focus, autoplay, codec, and remote-key problems.
- Stremio account, addon, catalog, subtitle, and streaming-server diagnostics.

## Boundaries

- Use only with a TV the user owns or is authorized to operate.
- Do not circumvent DRM or manipulate protected streaming services.
- Do not edit private Stremio, Netflix, DStv, or other app data without a supported API.
- Do not attempt shell, SSH, filesystem, firmware, bootloader, secure-boot, or UART write access.
- Do not expose media publicly or create a tunnel automatically.

## First Steps

1. Confirm the target TV and authorization.
2. Inspect the target project and set `VIDAA_PROJECT_ROOT` if needed.
3. Identify the Mac LAN address with `ipconfig getifaddr en1` or the active interface.
4. Use the TV's LAN URL, such as `http://192.168.1.82:8080/player.html`. `localhost` from the TV means the TV itself, not the Mac.
5. Prefer H.264/AAC MP4 for browser compatibility.

## Stremio Model

Stremio accounts synchronize addon configuration, catalogs, library state, and
watch progress. They do not store movie files and do not grant access to paid
services. A title appearing in Discover is not proof that a playable stream
exists.

Use this diagnostic sequence:

1. Confirm the TV bookmark is logged into the same Stremio account as the account where addons were installed.
2. Open a title's details page and distinguish metadata/catalog results from actual stream results.
3. If no streams appear, check addon installation, addon configuration, region availability, and the addon response.
4. If streams appear but playback returns to details, test the stream URL, codec, server requirement, and browser/native-player handoff.
5. Never treat Netflix, Paramount+, or another paid catalog addon as authorization to bypass that provider's official app or DRM.

Useful official resources:

- Stremio website and account: `https://www.stremio.com/`
- Official addon collection: `https://api.strem.io/addonscollection.json`
- Official OpenSubtitles addon source: `https://github.com/Stremio/stremio-opensubtitles`
- Official addon SDK: `https://github.com/Stremio/stremio-addon-sdk`
- Official addon publishing helper: `https://stremio.github.io/stremio-publish-addon/`
- Official catalog builder: `https://stremio.github.io/stremio-catalog-builder/`

The community list at `https://github.com/doingodswork/awesome-stremio` is a
discovery index, not a security or legality guarantee. Useful categories from
it include OpenSubtitles, Kitsu anime catalogs, IMDb list/tag catalogs, and
curated top-movie catalogs. Treat third-party stream, torrent, debrid,
download, DNS-spoofing, and account-proxy tools as untrusted until independently
reviewed. Do not install them automatically.

Catalog addons such as IMDb/Kitsu/top-movie lists help organize Discover but do
not necessarily return streams. A stream addon or an authorized owned-media
service is still required for playback.

## Account and Server Setup

There is no supported official Stremio CLI that safely logs into an account and
installs arbitrary addons or paid-service integrations. Prefer the Stremio web
interface, official addon install links, and the TV bookmark. Never automate
password capture, session-token extraction, or Netflix/Paramount credentials.

For authorized streams or owned media that need a server/transcoding, run an
official Stremio server or a supported media server on a LAN machine. Configure
the VIDAA build with:

```text
https://noobygains.github.io/stremio-vidaa-tv/?server=http://<mac-lan-ip>:11470
```

Verify the server from the TV's network, not only from the Mac. `localhost` and
`127.0.0.1` in the TV browser refer to the TV itself. The server must listen on
the LAN interface and the firewall must allow the selected port.

## Existing Project Commands

```sh
.venv/bin/python tools/tvctl.py pair       # vendor PIN flow; only when needed
.venv/bin/python tools/tvctl.py status
.venv/bin/python tools/tvctl.py info
.venv/bin/python tools/tvctl.py raw applist 30
.venv/bin/python tools/tvctl.py key KEY_HOME
.venv/bin/python tools/tvctl.py volume 25
.venv/bin/python tools/tvctl.py launch youtube
.venv/bin/python tools/tvctl.py launch-url http://<mac-lan-ip>:8080/player.html
.venv/bin/python tools/devserver.py 8080
```

## Helper Commands

```sh
VIDAA_PROJECT_ROOT=/path/to/project ./scripts/serve-media /path/to/project/app/media/demo.mp4
VIDAA_PROJECT_ROOT=/path/to/project ./scripts/tv-action status
VIDAA_PROJECT_ROOT=/path/to/project ./scripts/tv-action launch-url http://<mac-lan-ip>:8080/player.html
VIDAA_PROJECT_ROOT=/path/to/project ./scripts/verify-playback
```

## Pairing and Secrets

Pairing requires the PIN displayed by the TV. Pair tokens are persisted by the
existing project tool. Never print, copy, commit, or send token files, TLS keys,
or credentials. Treat telemetry and media filenames as potentially private.

## Streaming Workflow

1. Validate the media file and codec with `ffprobe` when available.
2. Stage it under `app/media/` only with explicit user approval; avoid copying large files unnecessarily.
3. Start `tools/devserver.py` on a LAN-accessible address and port.
4. Confirm the player returns `200` and the media endpoint returns `206 Partial Content` for a Range request.
5. Launch the player URL through `tvctl.py` or navigate manually in the TV browser.
6. A `boot` event is not enough. Successful playback requires `canplay`, `playing`, or advancing `timeupdate` events with `readyState=4` and no media error.
7. If MQTT key commands do not create browser key events, test muted autoplay first. This separates input-focus issues from media-decoding issues.

## Diagnostics

- Page loads but video does not: check the server process, LAN IP, media URL, MIME type, and Range response.
- `localhost` fails: replace it with the Mac's LAN IP.
- `launch-url` reports success but no page beacon arrives: navigate manually and inspect network reachability.
- Page beacon arrives but no media event: test H.264/AAC MP4 and inspect server logs.
- Video decodes but OK/arrows do nothing: the browser may consume remote keys; use autoplay for diagnosis and test character keys/focus.
- App configuration request: explain that MQTT can launch/control supported app surfaces but cannot edit private app storage.

## Stremio Troubleshooting

- Discover works but no Play/stream entries: metadata is available; configure a legitimate stream or owned-media addon.
- The TV shows different addons: sign into the same account and reload the bookmark; addon state is account-dependent.
- Stream list exists but playback fails: try a browser-compatible H.264/HEVC source, then verify the server URL and Range support.
- AV1 or unsupported source: use an authorized server/transcoder; the reference VIDAA browser may not decode AV1.
- Netflix or Paramount title: use the provider's official VIDAA app; Stremio metadata does not replace provider access.
- Anime catalog but no playback: a catalog such as Kitsu supplies metadata only; use an authorized playback source or the official anime service app.
