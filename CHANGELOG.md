# Changelog

## 1.2.4 — 2026-07-25

### Packaging
- Installer names include the OS for easier downloads:
  - `YTLiveRecorder-Windows-Setup.exe`
  - `YTLiveRecorder-Linux.AppImage`
  - `YTLiveRecorder-Linux.deb`

## 1.2.3 — 2026-07-25

### UI
- Channel example handle is `liquid_cryptid` (placeholder + hint)

## 1.2.2 — 2026-07-25

### Features / UX
- **Update badge** next to the title when a newer release is available (click to update)
- Silent update checks on startup and **every 4 hours** (badge only; no popup unless you click)

### Packaging
- Installer filenames are **versionless**: `YTLiveRecorder-Setup.exe`, `YTLiveRecorder.AppImage`, `YTLiveRecorder.deb`  
  (Version remains in the app, git tags, and `latest.yml`)

## 1.2.1 — 2026-07-25

### UI
- Add-channel label notes the **@ is optional**
- Removed the long cookies help blurb (dropdown only)

## 1.2.0 — 2026-07-25

### Features
- **In-app updates** via [electron-updater](https://www.electron.build/auto-update) + **public GitHub Releases** (free, works off your LAN).
  - Help → **Check for Updates…** / **Open Releases Page**
  - Quiet check a few seconds after startup (prompts only when a newer release exists)
  - Windows NSIS and Linux **AppImage** can download and install; `.deb` opens the releases page
- Docs: `docs/UPDATES.md`, `npm run release:github` publish helper

### Publish
- `build.publish` → `liquidcryptid/yt-live-recorder` on GitHub (public releases; Forgejo remains dev origin)
- Optional feed override: `YT_LIVE_RECORDER_UPDATE_URL` (generic provider)

## 1.1.8 — 2026-07-25

Stable Windows tester build after install-path, stop/finalize, logging, and temp-cleanup work. Linux shares the same app code.

### Fixes
- **Temp folder actually cleans on Windows:** after stop/move, force-kill orphan `yt-dlp`/`ffmpeg` processes that held `.part` files open (EBUSY), then `rmdir /s /q` with retries.
- **Per-recording temp session dirs** so locked leftovers cannot block or pollute the next run.
- **Startup temp purge** clears `%TEMP%\\YTLiveRecorderTemp` / `/tmp/YTLiveRecorderTemp` when the app launches.

### Verified (Windows smoke)
- NSIS → Program Files; tools stage to AppData; live record → Stop → clean `.mp4` on UNC share; `TEMP CLEANUP COMPLETE`; clean quit.

## 1.1.7 — 2026-07-25

### Fixes
- **No more `.part` destinations:** finished files are always saved with a clean name (`.mp4` / `.mkv`), even if yt-dlp left a `.part` source.
- **Pick newest media, not stale leftovers:** when multiple temp files exist, finalize chooses the newest complete-looking container.
- **Temp cleanup retries:** deletes channel temp after move (with retries); clears stale files before the next recording starts.

## 1.1.6 — 2026-07-25

### Fixes
- **Windows EBUSY on upgrade/restart:** staging tools into `%APPDATA%\\…\\bin` no longer aborts the app when `yt-dlp.exe` is locked by a leftover process. Keeps existing files, or runs tools from Program Files for that session.

## 1.1.5 — 2026-07-25

### Fixes
- **Stop Monitoring finalizes recordings:** stopping no longer abandons the yt-dlp process without merge/move. Kill → process partials (including `.part` leftovers) → copy to the target folder. Fixes Windows hang where Stop left media in temp and never moved it.
- **Non-blocking moves:** large/UNC copies use streams so the UI does not freeze during `finishMove`.
- **Harder Windows process kill:** taskkill force path for stuck yt-dlp/ffmpeg trees.

## 1.1.4 — 2026-07-25

### Fixes
- **Windows installer default path:** NSIS is per-machine (`perMachine: true`) so the default install location is under **Program Files** (requires elevation), not `%LOCALAPPDATA%\Programs`.

## 1.1.3 — 2026-07-25

### Fixes
- **No legacy log migration:** old flat logs (`~/yt-live-recorder.log`, etc.) are never copied into userData. That copy blocked AppImage first launch for ~14 minutes when a 32 GB log existed.
- **Tamer logging:** routine `NOT live` checks are debug-level; daily logs rotate at 25 MiB; **only the last 7 days** of log files are kept (older `*.log` under userData/logs are deleted on startup).
- **Clean exit on window close:** kill yt-dlp/ffmpeg children on stop and on `beforeunload`; hard `app.exit` if the process still hangs after the window is gone (AppImage zombie case).

## 1.1.2 — 2026-07-24

### Fixes
- **Persistent tools directory:** yt-dlp/ffmpeg/ffprobe are copied from the package into `~/.local/share/yt-live-recorder/bin` (Windows: under userData `bin/`) on startup. Runtime no longer depends on AppImage `/tmp/.mount_*`.
- **Temp is scratch-only:** `/tmp/YTLiveRecorderTemp` (or `%TEMP%\YTLiveRecorderTemp`) is for in-progress recordings and is safe to clear when the app is idle.

## 1.1.1 — 2026-07-24

### Fixes
- **AppImage/packaged spawn ENOTDIR:** renderer detects packaged mode without Electron `app` (main-only) and uses `resources/bin` paths from the main process for yt-dlp/ffmpeg.

## 1.1.0 — 2026-07-24

First packaged multi-platform release.

### Installers
- **Linux:** AppImage + `.deb` with bundled yt-dlp, ffmpeg, and ffprobe
- **Windows:** NSIS setup with bundled tools (+ MinGW `libwinpthread-1.dll`)
- Custom **LGPL** companion FFmpeg n7.1.1 (static OpenSSL) for linux-x64 and win-x64

### App
- Records best available YouTube quality (`bestvideo+bestaudio/best`) — no quality preset UI
- Cookie-aware yt-dlp clients (mweb/web_creator when cookies; android path without)
- Cookies UI: None / Firefox / Chrome / Edge (dropped Brave/Opera/Chromium)
- Settings/logs under Electron userData; recordings temp under OS `/tmp` (or `%TEMP%`)
- Help menu: Open Logs Folder, Open Current Log File, Licenses, About
- Unified main+renderer logging; tool stderr noise demoted to debug
- Graceful process kill (SIGINT → TERM → KILL; Windows taskkill tree)

### Packaging / legal scaffolding
- Vendor manifest, download-deps, prepare-resources, assert-vendor
- THIRD_PARTY_NOTICES, EULA template, LGPL source offer stubs

### Scope
- Linux x64 + Windows x64 only (no macOS)

## 1.0.0

Initial Electron app (source-only; system yt-dlp/ffmpeg).
