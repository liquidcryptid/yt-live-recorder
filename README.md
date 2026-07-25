# YTLiveRecorder

Desktop app that **watches YouTube channels** and **records livestreams** automatically (yt-dlp + ffmpeg).

> Not affiliated with, endorsed by, or sponsored by YouTube or Google.  
> You are responsible for complying with applicable laws and YouTube’s terms.

**Current version: 1.2.4**

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
7. Click **Stop Monitoring** when you want to stop. The app finalizes the file and moves it into  
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
| **Logs** | Help → **Open Logs Folder** (or the link at the bottom of the window) |
| **Where tools live** | Windows: `%APPDATA%\yt-live-recorder\bin` · Linux: `~/.local/share/yt-live-recorder/bin` |
| **Temp / scratch** | Safe to delete when the app is **not** recording: `%TEMP%\YTLiveRecorderTemp` or `/tmp/YTLiveRecorderTemp` |
| **Public lives** | Usually work with cookies = **None** |
| **Members-only lives** | Try Firefox/Chrome cookies; close the browser first |

---

## Patch notes

Full history: [`CHANGELOG.md`](CHANGELOG.md).

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

---

## For developers

### Dev setup

```bash
npm install
npm run download-deps              # yt-dlp into vendor/bin/<platform>

# Companion LGPL ffmpeg+ffprobe (Docker required)
npm run build:ffmpeg:linux && npm run install:ffmpeg:linux
# Windows cross-build from Linux:
npm run build:ffmpeg:win && npm run install:ffmpeg:win

npm start
# YT_LIVE_RECORDER_USE_SYSTEM_BINS=1 npm start
```

### Package installers

```bash
npm run pack:linux     # → dist/YTLiveRecorder-Linux.AppImage + .deb
npm run pack:win       # → dist/YTLiveRecorder-Windows-Setup.exe
npm run pack:all
```

### Publish a public update (GitHub)

```bash
npm run pack:all
npm run release:github   # gh auth login first; uploads installers + latest.yml
```

- **Development git host:** Forgejo on LAN (`git push origin`) — see local `origin`.
- **Public update CDN:** [GitHub Releases](https://github.com/liquidcryptid/yt-live-recorder/releases) only (source need not be pushed).
- Details: **`docs/UPDATES.md`**.

### Runtime paths

| What | Location |
|------|----------|
| App install (Windows) | `C:\Program Files\ytliverecorder\` |
| Tools | Linux: `~/.local/share/yt-live-recorder/bin` · Windows: `%APPDATA%\yt-live-recorder\bin` |
| Settings / logs | Linux: `~/.config/yt-live-recorder/` · Windows: `%APPDATA%\yt-live-recorder\` |
| Logs | `<userData>/logs/yt-live-recorder-YYYY-MM-DD.log` |
| Scratch | `/tmp/YTLiveRecorderTemp` or `%TEMP%\YTLiveRecorderTemp` |

### Legal / licensing (shipping)

- `docs/LEGAL.md`, `docs/EULA.md`, `docs/SOURCE_OFFER.txt`, `docs/RELEASE_LEGAL_CHECKLIST.md`
- `licenses/THIRD_PARTY_NOTICES.txt`

Application code: proprietary / all rights reserved unless otherwise stated. Third-party: see `licenses/`.
