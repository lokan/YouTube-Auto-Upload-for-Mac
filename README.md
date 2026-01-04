# YouTube Auto Upload for Mac

Automated YouTube uploads on macOS using a simple folder-based workflow.

This project is designed for creators, editors, and travelers who want to:
- export a video
- drop it into a folder
- walk away
- and let YouTube upload happen automatically.

---

## What this project IS

- A macOS workflow based on folders
- A shell + Python wrapper around `tokland/youtube-upload`
- Designed for real-world creator workflows (Final Cut, Compressor, travel setups)
- Fully local, no cloud services involved

---

## What this project is NOT

- ❌ Not a GUI application
- ❌ Not a YouTube client replacement
- ❌ Not a fork of `tokland/youtube-upload`
- ❌ Not cross-platform

---

## Requirements

- macOS
- Homebrew
- Python 3 (installed via Homebrew)
- A Google Cloud project with:
  - YouTube Data API v3 enabled
  - OAuth **Desktop App**
  - Your Google account added as a **test user**

---

## Folder structure (this matters)

This workflow relies on a fixed folder structure located in **Downloads**.

On your Mac, you will end up with:
