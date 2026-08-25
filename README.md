# YTLiveRecorder

Desktop app that **watches YouTube channels** and **records livestreams** automatically (yt-dlp + ffmpeg).

> Not affiliated with, endorsed by, or sponsored by YouTube or Google.  
> You are responsible for complying with applicable laws and YouTube’s terms.

<!-- Public / end-user doc. Maintainer notes: docs/DEV.md (Forgejo). Version is also in package.json. -->

**Current version: 1.3.3**

**Privacy:** [Privacy Policy](https://github.com/liquidcryptid/yt-live-recorder/blob/main/docs/PRIVACY.md) · Support: liquidcryptid@gmail.com

---

## Download

Get the latest build from **[GitHub Releases](https://github.com/liquidcryptid/yt-live-recorder/releases/latest)**.

| Your computer | Download this file |
|---------------|--------------------|
| **Windows** (most people) | `YTLiveRecorder-Windows-Setup.exe` |
| **Linux** | `YTLiveRecorder-Linux.AppImage` |

Other files on the release page (`latest.yml`, `*.blockmap`, checksums) are for **automatic updates** — you can ignore them for a first install.

---

## Usage

### Windows

1. Download **`YTLiveRecorder-Windows-Setup.exe`**.
2. Run it. If **SmartScreen** appears: **More info** → **Run anyway** (the installer is not code-signed yet).
3. Allow **administrator** access when prompted (installs under Program Files).
4. Launch **YTLiveRecorder** from the Start Menu or desktop shortcut.
5. In the app:
   1. **Browse** → choose a folder for finished recordings (local disk or network share).
   2. **Add channel** — type the channel handle (e.g. `liquid_cryptid`). The `@` is optional.
   3. Optionally check **Use cookies from Firefox** if you need membership/age-restricted streams. Sign in to YouTube in Firefox first; Firefox does not need to stay open.
   4. Click **Start Monitoring**.
6. When a channel goes live, recording starts automatically.
7. While a channel is **Recording**, click **Stop** next to that channel to save the current segment and keep monitoring. If the stream is still live, recording resumes automatically after the file is saved.
8. Click **Stop Monitoring** when you want to stop watching entirely. The app finalizes any active files and moves them into  
   `your-folder\ChannelName\…` as a normal `.mp4` (not a `.part` or `.mkv` file).

**Updates:** a green **Update x.y.z** badge may appear next to the title, or use **Help → Check for Updates…**.

### Linux (AppImage)

1. Download **`YTLiveRecorder-Linux.AppImage`**.
2. Make it executable and run:
   ```bash
   chmod +x YTLiveRecorder-Linux.AppImage
   ./YTLiveRecorder-Linux.AppImage
   ```
3. Same in-app steps as Windows (folder → add channels → Start Monitoring).

AppImage supports **in-app updates** the same way as Windows.

### Tips

| Topic | Detail |
|-------|--------|
| **Stop vs Stop Monitoring** | **Stop** (per channel) saves that download and keeps watching; if still live, the next segment starts from the **live edge** (not from the beginning again). **Stop Monitoring** ends detection for all channels |
| **Remove** | Removes the channel from the list; if it was recording, the current segment is still saved |
| **Closing the window (X)** | Saves active recordings first (please-wait popup), then exits |
| **Check interval** | Fixed at **20 seconds** (not configurable) |
| **Logs** | Help → **Open Logs Folder** — send this file if something fails |
| **Where tools live** | Windows: `%APPDATA%\yt-live-recorder\bin` · Linux: `~/.local/share/yt-live-recorder/bin` |
| **Temp / scratch** | Safe to delete when the app is **not** recording: `%LOCALAPPDATA%\yt-live-recorder\YTLiveRecorderTemp` or `~/.cache/yt-live-recorder/YTLiveRecorderTemp` (on disk, not `/tmp`). Closing the app can leave the last copied file there until the next launch (startup clears it). |
| **From the start** | Lives record from the beginning of YouTube’s rewind window when available; the row shows **Catch-up** (title, size/speed) then **LIVE** once rewind has reached the live edge. If the live ends before rewind is done, the **same file** keeps grabbing remaining DVR (**ENDED — finishing catch-up**). If yt-dlp drops while they are **still live**, it is restarted on the same file. If nothing is written for a minute after the live ended, the file is force-saved and the channel goes back to live detection. A new live is a **new** from-start recording. After **Stop**, the next segment is **LIVE** only |
| **Public lives** | Usually work with **Use cookies from Firefox** unchecked |
| **Members-only lives** | Check **Use cookies from Firefox**. Sign in to YouTube in Firefox; Firefox does not need to stay open. The app copies cookies on Start Monitoring and warns if they go out of date. |
| **File format** | Finished recordings are always **MP4** (editor-friendly; same container whether you Stop or the stream ends). |

---

## Patch notes

Full history: [`CHANGELOG.md`](CHANGELOG.md).

### 1.3.3

- After per-channel **Stop**, the new LIVE row shows size and download speed again (live-edge HLS).
- Stop only ends this app’s yt-dlp/ffmpeg for that segment (Windows and Linux). Other yt-dlp downloads on the same PC are left running.

### 1.3.2

- From-start is one yt-dlp job, like the CLI (`@handle/live`). A connection drop restarts it on the **same file**. If the live ends during catch-up, that same file keeps going until DVR is done. After the file is saved, the channel goes back to live detection — a later live is a new Catch-up.
- The Catch-up row becomes **LIVE** only when both video and audio are at the DVR head (audio often finishes rewind first).
- If save/remux fails (for example the disk is full), leftover parts are left in the temp folder instead of being deleted.
- Cookie-client JS challenges use the Electron binary as Node (same as the installer). A local Deno install is not used.
- In-progress recordings scratch the disk cache (`~/.cache/yt-live-recorder` / `%LOCALAPPDATA%\yt-live-recorder`), not `/tmp` (often a RAM disk).

### 1.3.1

- Multi-hour from-start recordings keep their full length when the stream ends (ffmpeg remux no longer stops at the first video/audio timestamp gap).
- A hung download after the row already says **LIVE** is saved and continued from the live edge (same ~75s stall as catch-up).
- An in-progress yt-dlp merge (`.temp.mp4`) is no longer published as the finished file.

### 1.3.0

- Recordings, cookies, and live checks run in the main process. The window no longer has Node access (safer if a page is ever compromised). Closing the app still saves files if the UI crashes.
- Lives start from the beginning of YouTube’s rewind window when available (**Catch-up** then **LIVE**). Finished files are always **MP4**.
- Catch-up keeps going after a live ends (same file; resumes from the VOD if the live URL dies). Faster rewind; the row switches to **LIVE** even without a fragment total. No 15-minute wait if they come back.
- **Use cookies from Firefox** for members-only / age-restricted streams. Firefox does not need to stay open.
- A live channel that briefly has “no formats” is retried in ~30 seconds (Start Monitoring also retries immediately).

### 1.2.11

- App window and Microsoft Store tiles use the YTLiveRecorder icon (not the default Electron artwork).

### 1.2.10

- **Long-run stability** — live checks (page probe + status) run in the main process instead of the UI, fixing a Chromium file-descriptor / shared-memory leak that grew over hours of monitoring many channels.

### 1.2.9

- Recording filenames use your computer’s local time (not UTC) in the date prefix.

### 1.2.8

- Fixed scheduled/upcoming streams flashing **Recording** then idle (no formats yet).
- Cooldown channels no longer re-check YouTube every 20s.

### 1.2.7

- Faster live detection (page probe + yt-dlp only when needed); fixed **20s** check interval.
- Closing the app (X) waits until recordings are stopped and moved to your folder.
- Cleaner file names; best quality live formats; updated bundled **yt-dlp**.
- Cookies note and UI polish; more readable logs for troubleshooting.

### 1.2.6

- **Stop** button next to each channel that is downloading — saves that segment to your recordings folder.
- Monitoring continues after a per-channel Stop; a new segment starts automatically if the channel is still live.
- **Stop Monitoring** still fully stops watching and finalizes everything.

### 1.2.5

- Logs split by day while the app stays open overnight.
- Stopping one channel no longer kills recordings on other channels.
- Fixed channels getting stuck “recording” after a short restart failure.
- Less log spam from routine HLS CDN messages; 10‑minute cooldown when yt-dlp finds no formats.

### 1.2.4

- Installer filenames include the OS (`Windows` / `Linux`) so downloads are easier to pick.

### 1.2.3

- Channel example uses `liquid_cryptid`.

### 1.2.2

- Green **update badge** next to the title when a new version is available (click to update).
- Silent update checks after startup and every **4 hours** (badge only; no popup spam).

### 1.2.1

- Add-channel text notes that **@ is optional**.
- Removed the long cookies help paragraph (dropdown kept).

### 1.2.0

- **In-app updates** from GitHub Releases (Help → Check for Updates).
- Windows NSIS + Linux AppImage support auto-install; `.deb` is manual.

### Earlier (1.1.x highlights)

- Reliable **Stop Monitoring** → finalize and move to your recordings folder.
- Clean filenames (no `.part` on the finished file).
- Temp folder cleanup on Windows; Program Files install path.
- Quieter logging and 7-day log retention.
