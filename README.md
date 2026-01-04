# YouTube Auto Upload for Mac

Automatically upload Final Cut Pro or Compressor exports to YouTube on macOS.

This project provides a **robust watch-folder workflow** built with:
- a hardened shell script,
- macOS Folder Actions (Automator),
- and the `tokland/youtube-upload` Python client.

Drop a video export into a folder, close your laptop, and let the upload start automatically **once the final file is actually ready**.

---

## Why this exists

If you export videos while traveling (events, CES, conferences, hotels, weak Wi-Fi), waiting in front of your Mac for an export to finish is a waste of time.

This workflow solves common real-world issues:

- Final Cut Pro / Compressor create **temporary or segmented files**
- Folder Actions trigger **too early**
- Videos get uploaded **twice**
- Partial files are sent by mistake

This setup avoids all of that.

---

## Features

- Watches a folder (`INBOX`) for new exports
- Ignores temporary / segmented files
- Waits until the file size is **stable for 60 seconds**
- Prevents double uploads with a lock mechanism
- Automatically moves files to:
  - `DONE` on success
  - `FAILED` on error
- Writes one log file per video
- Uses OAuth (no password storage)

---

## Folder structure

```text
YouTube Auto Upload/
├── 0 - ADMIN     # script, Python venv, OAuth client
├── 1 - INBOX     # export destination (watched folder)
├── 2 - DONE      # successfully uploaded videos
├── 3 - FAILED    # failed uploads
└── 4 - LOGS      # per-video logs + locks
