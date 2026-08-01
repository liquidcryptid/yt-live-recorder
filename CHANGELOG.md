# Changelog

## 1.2.10 — 2026-08-01

### Bug fixes
- **Long-run Chromium FD / shared-memory leak while monitoring** — live probes and yt-dlp status checks no longer run inside the UI renderer. Multi-MB YouTube page fetches used Chromium Mojo shared memory and left deleted `/dev/shm` FDs growing for hours (often ~1k+ FDs after a day of 20s polls). Probes now use Node `fetch` + a queued yt-dlp status print in the **main process** (`lib/live-check.js`); the renderer only receives yes/no via IPC. Recording spawn is unchanged for this release (still `nodeIntegration` in the UI process).

### Reliability
- Probe HTML body capped at 3 MiB in main (markers appear earlier; less peak memory)
- Live-check helper processes are killed from main on Stop Monitoring and quit

### Notes
- After upgrading, multi-hour monitoring should keep the UI process open file-descriptor count roughly flat (dozens), not climbing into the thousands
- Concurrent recordings still use ~100 MB RAM each (yt-dlp + ffmpeg); that is expected, not this leak

## 1.2.9 — 2026-07-31

### Bug fixes
- **Recording filenames use local time** — the `YYYY-MM-DD-HH-MM` prefix is the computer’s wall-clock time (not UTC), so evening streams no longer get the next day’s date when the machine is west of UTC

## 1.2.8 — 2026-07-31

### Bug fixes
- **Scheduled / upcoming streams no longer flash Recording then idle** — pages like `@channel/live` for a not-yet-started stream often still contain `"style":"LIVE"` and `"isLive":true` while `"isLiveNow":false` / `"isUpcoming":true`. The HTTP probe no longer treats weak LIVE chrome as live when the stream is offline-now, and yt-dlp `is_upcoming` is treated as not live (so we do not start a download that immediately fails with “No video formats found”).
- **Cooldown skips the live check** — channels waiting out a no-formats / error cooldown no longer re-run probe + yt-dlp every 20s (was spamming “starting record anyway” in the log).

## 1.2.7 — 2026-07-31

### Features
- **Faster live detection** — two-phase check: cheap page probe first, then yt-dlp only when needed; fixed **20s** check interval (no longer user-configurable)
- **Smarter end detection** while recording — probe first; full yt-dlp only when the page looks offline (less hammering YouTube mid-stream)
- **Reliable app close (X)** — shows a please-wait status window, stops downloads, moves files to your recordings folder, then exits
- **Best quality lives** — `bestvideo*+bestaudio/best` with format sorting suited to live HLS; bundled **yt-dlp 2026.07.23** nightly for current YouTube
- **Cleaner filenames** — `YYYY-MM-DD-HH-MM - Stream Title.mkv` (strips YouTube’s trailing clock from titles; no random suffix)

### UX
- Cookies hint: only needed for members-only / age-restricted streams
- Separator between settings and Start/Stop Monitoring
- **Remove** channel mid-recording still saves the segment, then drops that channel (no auto-resume)
- Progress line shows the file name only (not full `Destination:` path)

### Reliability
- Hybrid save path so Stop/close does not double-copy the same segment to the NAS
- Live-check queue, monitor re-entrancy guard, log prune on day roll, clearer WARN/ERROR text for testers
- FFmpeg live HLS fix (`allowed_extensions`) for current stream URLs

### Notes
- Prefer **Stop Monitoring** or wait for the close status window to finish before force-killing the process
- If a save fails, use Help → **Open Logs Folder** and look for WARN/ERROR lines

## 1.2.6 — 2026-07-31

### Features
- **Per-channel Stop button** next to streams that are actively recording — stops only that download, finalizes/merges, and moves the file into your destination folder
- **Monitoring stays on** after a per-channel Stop (unlike **Stop Monitoring**, which ends detection for every channel)
- **Automatic resume** when the channel is still live: a new recording segment starts as soon as finalize finishes (no need to wait for the next check interval)

### Notes
- Use per-channel **Stop** to split long streams into segments or free disk/temp mid-broadcast
- Use **Stop Monitoring** when you want to fully stop watching and finalize everything

## 1.2.5 — 2026-07-27

### Bug fixes
- **Daily log files** now roll at UTC midnight while the app stays open (previously the day stamp was fixed at process start, so multi-day runs stayed on e.g. `…-2026-07-25.log`)
- **Stopping one channel no longer kills every other download** — global `pkill`/`taskkill /IM` of yt-dlp/ffmpeg is skipped while siblings are still recording (this was aborting MLCPodcast mid-stream when another channel auto-ended)
- **Stuck “recording” after a failed restart** — dead process slots and the 60s finalize lock no longer block re-recording the same channel
- Routine HLS CDN noise (`keepalive request failed`, `Cannot reuse HTTP connection`, …) is debug-level again so real problems are visible
- **No video formats found** applies a 10-minute cooldown (stops hammering channels like false-positive lives every 30s)

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
