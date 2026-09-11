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
