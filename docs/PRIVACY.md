# Privacy Policy — YTLiveRecorder

**Last updated:** 2026-08-12  
**Publisher:** Liquid Cryptid  
**Contact:** liquidcryptid@gmail.com  
**Product:** YTLiveRecorder (including the Microsoft Store edition)

This policy describes how the YTLiveRecorder desktop application handles information. It is written for end users and for store listing requirements. It is not legal advice.

## Summary

- YTLiveRecorder runs **on your computer**.
- Recordings, settings, and logs stay **on your device** by default.
- The app does **not** require an account with Liquid Cryptid.
- The app does **not** sell your personal data.
- Optional features may contact **third-party services you choose** (for example YouTube, or Buy Me a Coffee if you donate).

## Who we are

YTLiveRecorder is published by **Liquid Cryptid** (“we”, “us”).  
Support and privacy questions: **liquidcryptid@gmail.com**.

## What the app does

YTLiveRecorder monitors YouTube channel pages or streams that **you** configure and can record livestreams to folders **you** choose, using bundled open-source tools (including yt-dlp and FFmpeg).

## Information stored on your device

The app may create or use local files such as:

| Data | Typical location (Windows) | Purpose |
|------|----------------------------|---------|
| Settings (folder path, channel list, cookie browser preference) | `%APPDATA%\yt-live-recorder\settings.json` | Remember your preferences |
| Application logs | `%APPDATA%\yt-live-recorder\logs\` | Diagnose problems |
| Staged tools (yt-dlp, ffmpeg copies) | `%APPDATA%\yt-live-recorder\bin\` | Run recordings reliably |
| Temporary recording segments | `%TEMP%\YTLiveRecorderTemp\` | In-progress downloads before finalize |
| Finished recordings | The folder **you** select | Your media files |
| Optional cookies export | Under app data (if you enable browser cookies) | Access membership/age-restricted streams **you** are allowed to view |

We do not operate a Liquid Cryptid cloud service that collects these files from your PC.

## Information sent over the network

Depending on how you use the app, it may contact:

1. **YouTube / Google infrastructure**  
   To check whether a channel is live and to download stream media for channels you added. Traffic content and any authentication material are between your device and those services under **their** terms and privacy policies.

2. **Microsoft Store / Windows** (Store edition only)  
   Install, license, and update traffic is handled by Microsoft according to Microsoft’s policies.

3. **Buy Me a Coffee** (optional)  
   If you click **Donate**, your browser opens an external donation page. Payment and account data are handled by Buy Me a Coffee and their payment processors—not by YTLiveRecorder collecting card numbers inside the app.

4. **GitHub Releases** (non-Store builds only)  
   Optional automatic update checks may contact GitHub to see if a newer installer exists. The Microsoft Store edition **does not** use GitHub auto-update.

The app does not include third-party advertising SDKs or analytics SDKs for marketing profiles.

## Optional browser cookies

If you check **Use cookies from Firefox**, the app copies Firefox cookie data into a local Netscape file under app data (`cookies.from-browser.txt`) so recordings can use your logged-in YouTube session (for example members-only or age-restricted content). The copy is refreshed when you Start Monitoring. Unchecking the box deletes that file. You can instead place your own `cookies.txt` in the same folder.

- Only enable this if you understand that stream access uses **your** Firefox YouTube session.  
- You must be signed in to YouTube in Firefox; Firefox does not need to stay open.  
- You are responsible for complying with YouTube’s terms and applicable law.

## Children

YTLiveRecorder is a general-purpose desktop utility. It is not directed at children under 13. Do not use the app to collect personal information from children.

## Permissions and access

On Windows, the app needs access to:

- The **recordings folder** you choose  
- Local app data and temp folders it creates  
- Network access to fetch live status and stream data  
- Optionally, Firefox cookie stores if you enable **Use cookies from Firefox**  

It does not require you to create a Liquid Cryptid account.

## Data retention and deletion

- Delete recordings by removing files from your chosen folder.  
- Clear temp data when the app is **not** recording (`%TEMP%\YTLiveRecorderTemp`).  
- Browser cookie copies live in `%APPDATA%\yt-live-recorder\cookies.from-browser.txt`; uncheck **Use cookies from Firefox** or delete that folder to remove them.  
- Uninstall the app via Windows / Microsoft Store; you may also delete `%APPDATA%\yt-live-recorder` to remove settings, logs, cookies, and staged tools.

## Third-party services

YouTube, Google, Microsoft, Buy Me a Coffee, GitHub, and other services you interact with have their own privacy policies. This policy only covers YTLiveRecorder as published by Liquid Cryptid.

## Not affiliated with YouTube or Google

YTLiveRecorder is an independent product. It is **not affiliated with, endorsed by, or sponsored by YouTube or Google LLC**.

## Changes

We may update this policy when the product or practices change. The “Last updated” date at the top will change. Continued use after updates means you accept the revised policy for that version of the app.

## Contact

Privacy or support: **liquidcryptid@gmail.com**  
Project / releases: https://github.com/liquidcryptid/yt-live-recorder  
Microsoft Store product ID: **9MVWBRQ53RNF**
