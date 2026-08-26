# Changelog

## 1.3.4 — 2026-08-26

### Bug fixes
- **Stream-end save is not dropped while the monitor loop is running** — yt-dlp exit 1 (live ended, `Did not get any data blocks`) starts an async “still live?” check before save. `hasActiveRecording` treated the dead process as a stale slot and deleted it, so the close handler returned without copying the MP4. The check is now read-only; only save/retry paths clear the slot.
- **Help → About no longer opens the logs folder when you close the window with X** — Windows maps the title-bar close button to cancel. That was hitting **Open Logs Folder** (the second button). Esc and X now just dismiss the dialog.
- **Log timestamps and daily log files use local time** — line stamps and `yt-live-recorder-YYYY-MM-DD.log` follow the computer’s clock (and roll at local midnight), not UTC. Recording filename prefixes were already local.

## 1.3.3 — 2026-08-25

### Bug fixes
- **LIVE size and speed after per-channel Stop** — Stop restarts from the live edge (`--no-live-from-start`). The 1s size/speed ticker only sampled `--live-from-start` jobs, so the LIVE row came back with no KiB/s. The ticker now samples every active recording; live-edge segments are LIVE from the first byte and use disk size/rate (HLS/ffmpeg rarely fills yt-dlp `progress.speed`).
- **Stop leaves other people’s yt-dlp alone** — never `taskkill /IM` / `pkill -x`. Stop kills the spawned PID tree, then only yt-dlp/ffmpeg/ffprobe whose command line includes this app’s `YTLiveRecorderTemp` scratch and that segment’s `rec-…` folder. PyInstaller can exit the wrapper while a child keeps writing; that leftover is now stopped so remux is not EBUSY. A yt-dlp you started yourself is not matched.
- **Linux Stop leftover kill matches Windows** — the bundled yt-dlp is also PyInstaller: SIGKILL of the wrapper left the child running (reparented to init). Stop/resume snapshot-kill that tree (Linux `/T`) and session-sweep leftovers. Distro `python3 …/yt-dlp` is matched too; unrelated yt-dlp is not.

## 1.3.2 — 2026-08-24

### Changes
- **Let yt-dlp run the live** — one CLI-like `--live-from-start --continue` job on `@handle/live`. Catch-up vs LIVE is a UI label. A connection drop (yt-dlp dies while `/live` is still up) restarts yt-dlp on the **same file**. If the live ends during catch-up, that same file keeps grabbing remaining DVR (UI: **ENDED — finishing catch-up**). When the live has ended and nothing is written for **60 seconds**, the file is saved and the channel goes back on the 20s live-detect loop. A new live is a new from-start job. Per-channel Stop still splits and resumes from the live edge. Cookies unchanged.
- **From-start args match a working CLI** — `bestvideo[height<=?1080]+bestaudio/best`, no `-N 8`, no forced native dash, no `formats=missing_pot` unless Firefox cookies are on. Downloads always start at `@handle/live`.
- **JS runtime matches the installer** — Deno is not used even if it is on PATH (`npm start` used to pick a local Deno). yt-dlp gets `--js-runtimes node:` with the Electron binary as Node, same as a packaged build. Public lives do not need it.
- **Recording scratch is on disk cache, not `/tmp`** — Linux `/tmp` is often a small RAM disk (tmpfs). Overnight lives filled it (`Errno 28`) while the SSD still had space. Scratch is now `~/.cache/yt-live-recorder/YTLiveRecorderTemp` (Windows: `%LOCALAPPDATA%\yt-live-recorder\YTLiveRecorderTemp`). Old `/tmp/YTLiveRecorderTemp` is not auto-deleted.

### Bug fixes
- **Channel stop/restart is not treated as a connection drop** — `/live` offline → wait for yt-dlp (force-save if nothing is written for 60s) → move the file → live detection. If catch-up is still unfinished and yt-dlp drops, `--continue` that video’s VOD (`watch?v=`), not `/live` (which may already be a new stream). Connection-drop resume (`@handle/live`, same file) only runs while the channel is **still live**.
- **LIVE label waits for both video and audio** — YouTube can report `109/3071` on video while audio is already `3575/3575`. Using the lagging total kept the row on Catch-up at live bitrate. Both `.ytdl` streams within 3 fragments of their totals (or fragment index at live-edge rate for ~12s) flips to LIVE.
- **Failed remux no longer deletes the recording** — if ffmpeg cannot produce MP4 (disk full, etc.), leftover parts stay in the temp folder instead of being deleted.
- **On save, extra audio is trimmed if video died early** — leftover remux of split DASH can have hours of AAC after VP9 stopped. If audio is ≥15 minutes longer than a real video duration, audio is cut to the video end. Leftover `-FragN` files are joined when the `.part` is empty.

## 1.3.1 — 2026-08-23

### Bug fixes
- **Long from-start recordings are no longer truncated on save** — killing yt-dlp at stream end leaves `.f248.mp4.part` + `.f140.mp4.part`. ffmpeg remux used `-shortest` and `discardcorrupt`, so a YouTube DASH timestamp gap (common on multi-hour lives) stopped the copy at the first A/V overlap. Overnight jobs saved minutes (or seconds) of an 8-hour stream. Remux now copies the full parts (`-max_interleave_delta 0`, no `-shortest`) and retries a two-pass remux if the output is much smaller than the inputs. Auto-end waits ~20s for yt-dlp to close those parts (was 5s).
- **LIVE DASH hang after catch-up was invisible** — the ~75s “no new bytes” failover only ran during catch-up. Once the row said LIVE, a hung googlevideo URL sat there until `/live` went offline, then remuxed whatever tiny prefix existed. The same stall now saves and switches to the live edge. Size is logged every 5 minutes while LIVE so this shows up in the log.
- **In-progress yt-dlp merge is not treated as a finished file** — `title.temp.mp4` has no `moov` yet (killed mid-merge). Finalize skipped the real `.fXXX` parts and published a broken `.temp.mp4`. Those temp merges are ignored; leftovers are remuxed instead.

## 1.3.0 — 2026-08-22

### Security
- **UI process is isolated** — yt-dlp/ffmpeg spawn, kill, and file save run in the main process. The window uses `nodeIntegration: false`, `contextIsolation: true`, a sandboxed preload, and a frozen `window.ytlr` allowlist (no free-form argv, no temp-folder access from the page). Closing the app still saves recordings if the UI process has crashed. Closing/reloading the window no longer kills other programs’ yt-dlp (unchanged: PID tree only).

### Tools
- **yt-dlp nightly 2026.08.20.234504** (was 2026.07.23.234303)

### Features
- **YouTube `--live-from-start`** — recordings start from the beginning of the stream (YouTube DVR) using yt-dlp’s native dash/m3u8 downloader (fixed upstream 2026.03.17 / #16254, in our 2026.08.20 nightly). If from-start is unavailable, real 403s before any bytes, or “no data blocks” at start, the app retries from the live edge. Per-channel **Stop** always resumes from the live edge. While catching up, auto-end does not kill the job just because `/live` looks offline (stalls after 3 minutes still save).
- **Finished files are always `.mp4`** — Stop, stream-end, and a clean yt-dlp merge used to leave `.mkv`, `.mp4`, or `.f247.mkv` depending on how the job ended. Every published file is now MP4 (`--merge-output-format mp4` / `--remux-video mp4`, ffmpeg remux of leftovers with `+faststart`). Premiere, Resolve, Final Cut, and similar NLEs import MP4; MKV is a common reject. Codecs are still whatever YouTube served (often VP9/Opus or H.264/AAC); this is a remux, not a re-encode.
- **Per-channel Stop no longer saves a broken `.f247.mkv`** — from-start writes separate video/audio fragments; a hard kill left a truncated video-only `.part`. Stop now asks yt-dlp to exit (Windows: `taskkill` without `/F`, 8s grace) then ffmpeg remuxes leftovers (`-c copy`, discardcorrupt, `+faststart`) into a closed `.mp4` with audio when the audio part exists.
- **YouTube player clients** — without cookies, yt-dlp’s **default** clients (`formats=missing_pot` only). With Firefox cookies: `tv,web_safari,mweb,web_creator`. Forcing `android,ios,tv` on public lives skipped SABR URLs. A PO-token warning that *mentions* HTTP 403 is no longer treated as a from-start crash.
- **Firefox cookies checkbox** — Chrome/Edge cookie options are removed. Check **Use cookies from Firefox** for members-only / age-restricted streams. You must be signed in to YouTube in Firefox; Firefox does not need to stay open. On Start Monitoring the app copies Firefox cookies to `%APPDATA%\yt-live-recorder\cookies.from-browser.txt`. A warning appears if those cookies are missing or out of date (open Firefox, sign in, Start Monitoring again).
- **Download status shows stream title, size, and speed** — Catch-up / LIVE rows show the stream name with downloaded size and rate (e.g. `123.6 MiB · 76.3 KiB/s`). Fragment `current/total` is not shown (yt-dlp’s `.ytdl` index only moves when a DASH fragment *finishes*, so it looked stuck while the file was still growing). Catch-up closes and the row is **LIVE** once DVR catch-up finishes. After Stop, the next segment is **LIVE** only.
- **Catch-up continues after the live ends** — if `/live` goes offline while DVR catch-up is still running, the job keeps grabbing remaining fragments (UI: **ENDED — finishing catch-up**). Auto-stop only if the VOD is private/removed/members-only, or catch-up stalls for 3 minutes with no new fragments.
- **Stuck catch-up fails over to LIVE** — only if the fragment **index and disk size** are frozen ~75s (a real googlevideo hang). `.ytdl` `current_fragment.index` does not move until a DASH fragment *finishes*; 1080p frags can take over a minute while the `.part` still grows, which used to look stuck and abort a healthy catch-up.
- **Catch-up downloads 8 DASH fragments at a time** (`--concurrent-fragments 8`). Do not pass `--throttled-rate` (that aborts slow DVR fragments). Live-edge stays sequential.
- **Live channel no longer sits idle for 10 minutes after “No video formats found”** — if live-check already confirmed the stream, a miss retries in ~30s (Start Monitoring also clears that cooldown). Age-restricted lives wait 5 minutes and show a cookies hint instead of looping.
- **Public lives do not need Firefox** — cookies stay opt-in for members-only / age-restricted only. Without cookies the app uses yt-dlp’s **default** clients (visionos + webpage), which return 1080 DVR (`399+140` / `248+140`). Forcing `android,ios,tv` skipped SABR URLs and broke catch-up on every public live. Age-restricted lives still need Firefox cookies — the row shows **AGE RESTRICTED: REQUIRES COOKIES** in red. Checking that box retries only those blocked streams and never stops recordings that are already working.

### Bug fixes
- **Catch-up no longer aborts when the live ends** — the ~75s DASH-hang failover (save + switch to live edge) was firing after `/live` went offline (log: `Did not get any data blocks` → “continuing catch-up of remaining DVR” → then “stuck — switching to LIVE”). There is no live edge then, so the partial file was saved and catch-up stopped. Failover now re-checks liveness; if the stream has ended, the job keeps going and, if the DVR URL is dead, resumes the **same file** from the VOD (`watch?v=`). Auto-save still happens if the VOD is gone or nothing arrives for 3 minutes.
- **Catch-up matches CLI `--live-from-start` speed** — dropped `--throttled-rate 100K` (that flag is not a cap; it aborts a DASH fragment after 3s below 100 KiB/s and re-extracts the live, which is a VOD HTTP workaround). Slow DVR catch-up sat under 100 KiB/s for a long stretch, so the tripwire kept interrupting. Concurrent fragments for catch-up went from 3 to 8. Live-edge recordings stay sequential. There is still no `--limit-rate`.
- **Catch-up row switches to LIVE without a fragment total** — YouTube live DVR almost never reports `fragment_count`, so the old “within 3 fragments of total for 5s” check never fired. A catch-up that had already reached live cadence (~15 frags / 30s, live bitrate) stayed on the Catch-up row. When the count is missing, catch-up is treated as done if fragments arrive at live-edge rate (not a burst) for ~12s.
- **No 15-minute sit-out after a stream ends** — auto-end used to set a 900s cooldown, so a channel that dropped and came right back (tech issue, BRB) was ignored. After save, the next 20s check (or the immediate resume probe) can start a new file if they are live again.
- **Stop Monitoring / stream end no longer kill other programs’ yt-dlp** — 1.2.5 stopped `taskkill /IM` while *this* app still had sibling recordings, but when the last channel in the app stopped (or on startup temp purge) it still ran `taskkill /IM yt-dlp.exe` / `pkill -x yt-dlp`, which aborted every yt-dlp on the machine. Stop now kills only that job’s PID tree; leftover lock cleanup only targets processes whose command line includes `YTLiveRecorderTemp`.
- **Cookie file on the live probe hid live channels** — sending `.google.com` cookies to `youtube.com/@handle/live` redirected to Google `CookieMismatch` / sign-in, which has no LIVE markers, so the probe reported not live and never ran yt-dlp (e.g. @SekturVision). Probe Cookie header is YouTube-only; sign-in pages fall back to yt-dlp.
- **Chrome/Edge cookies no longer abort recordings** — `--cookies-from-browser` on Windows often fails with “Could not copy Chrome cookie database” (browser lock / Chromium encryption). The app now detects that, records without browser cookies, and retries immediately so public lives still save. Members-only still needs Firefox, a closed browser, or `cookies.txt`.
- **Firefox cookies recorded no formats** — cookie-safe YouTube clients need yt-dlp’s JS challenge solver. The app now passes `--js-runtimes node:<path>` (system Node, or Electron as Node) so n-challenge solving works and members/restricted lives can get playable formats.

### Reliability
- Recording logs now include extractor clients, yt-dlp pid, YouTube video id, cooldown remaining, and a warning when live-check says LIVE but yt-dlp listed no formats (so a silent sit-out is visible in Help → Open Logs Folder).

## 1.2.11 — 2026-08-12

### Packaging
- **Microsoft Store AppX tiles use the YTLiveRecorder icon** — `build/appx/` is generated from `build/icon.png` so electron-builder no longer ships the default Electron sample logos (Store certification 10.1.1)
- Window and packaged resources now include the app icon (no Electron atom in the title bar)

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
- `build.publish` → `liquidcryptid/yt-live-recorder` on GitHub (public releases)
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
