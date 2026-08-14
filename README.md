# YTLiveRecorder

Desktop app that **watches YouTube channels** and **records livestreams** automatically (yt-dlp + ffmpeg).

> Not affiliated with, endorsed by, or sponsored by YouTube or Google.  
> You are responsible for complying with applicable laws and YouTube’s terms.

<!-- Public / end-user doc. Maintainer notes: docs/DEV.md (Forgejo). Version is also in package.json. -->

**Current version: 1.2.11**

**Privacy:** [Privacy Policy](docs/PRIVACY.md) · Support: liquidcryptid@gmail.com

---

## Download

Get the latest build from **[GitHub Releases](https://github.com/liquidcryptid/yt-live-recorder/releases/latest)**.

| Your computer | Download this file |
|---------------|--------------------|
| **Windows** (most people) | `YTLiveRecorder-Windows-Setup.exe` |
| **Linux** (recommended) | `YTLiveRecorder-Linux.AppImage` |
| **Linux** (package install) | `YTLiveRecorder-Linux.deb` |

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
   3. Optionally set **Cookies from browser** if you need membership/age-restricted streams (close that browser first).
   4. Click **Start Monitoring**.
6. When a channel goes live, recording starts automatically.
7. While a channel is **Recording**, click **Stop** next to that channel to save the current segment and keep monitoring. If the stream is still live, recording resumes automatically after the file is saved.
8. Click **Stop Monitoring** when you want to stop watching entirely. The app finalizes any active files and moves them into  
   `your-folder\ChannelName\…` as a normal `.mp4` / `.mkv` (not a `.part` file).

**Updates:** a green **Update x.y.z** badge may appear next to the title, or use **Help → Check for Updates…**.

### Linux (AppImage — recommended)

1. Download **`YTLiveRecorder-Linux.AppImage`**.
2. Make it executable and run:
   ```bash
   chmod +x YTLiveRecorder-Linux.AppImage
   ./YTLiveRecorder-Linux.AppImage
   ```
3. Same in-app steps as Windows (folder → add channels → Start Monitoring).

AppImage supports **in-app updates** the same way as Windows.

### Linux (.deb)

```bash
sudo dpkg -i YTLiveRecorder-Linux.deb
# if needed:
sudo apt-get install -f
```

`.deb` installs do **not** use the in-app installer updater; download a new `.deb` from Releases, or switch to the AppImage for auto-update.

### Tips

| Topic | Detail |
|-------|--------|
| **Stop vs Stop Monitoring** | **Stop** (per channel) saves that download and keeps watching; **Stop Monitoring** ends detection for all channels |
| **Remove** | Removes the channel from the list; if it was recording, the current segment is still saved |
| **Closing the window (X)** | Saves active recordings first (please-wait popup), then exits |
| **Check interval** | Fixed at **20 seconds** (not configurable) |
| **Logs** | Help → **Open Logs Folder** — send this file if something fails |
| **Where tools live** | Windows: `%APPDATA%\yt-live-recorder\bin` · Linux: `~/.local/share/yt-live-recorder/bin` |
| **Temp / scratch** | Safe to delete when the app is **not** recording: `%TEMP%\YTLiveRecorderTemp` or `/tmp/YTLiveRecorderTemp` |
| **Public lives** | Usually work with cookies = **None** |
| **Members-only lives** | Try Firefox/Chrome cookies; close the browser first |

---

## Patch notes

Full history: [`CHANGELOG.md`](CHANGELOG.md).

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
