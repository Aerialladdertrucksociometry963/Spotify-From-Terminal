# spt — Spotify TUI (Python) 🎧🐍

---

[![License: MIT](https://img.shields.io/badge/license-MIT-blueviolet.svg)](https://opensource.org/licenses/MIT) [![Python Version](https://img.shields.io/badge/python-3.10%2B-blue)](https://www.python.org/) [![Release](https://img.shields.io/github/v/release/naarvent/Spotify-From-Terminal?label=last%20release)](https://github.com/naarvent/Spotify-From-Terminal/releases/tag/v2.0) [![Issues](https://img.shields.io/github/issues/naarvent/Spotify-From-Terminal)](https://github.com/naarvent/Spotify-From-Terminal/issues) [![GitHub stars](https://img.shields.io/github/stars/naarvent/Spotify-From-Terminal?style=social)](https://github.com/naarvent/Spotify-From-Terminal/stargazers)


---
A terminal-based Spotify client built with Python, Spotipy and Textual. It provides a keyboard-driven TUI for searching, browsing and controlling Spotify playback, viewing synced lyrics when available, managing playlists and devices, and more.


---

## Table of Contents 📚

- [Summary](#summary-)
- [Features ✨](#features-)
- [Requirements & Installation 🛠️](#requirements--installation-️)
- [Environment & Credentials 🔐](#environment--credentials-)
- [How it works (high level) ⚙️](#how-it-works-high-level-️)
- [APIs & External Services 🌐](#apis--external-services-)
- [Keybindings / Controls ⌨️](#keybindings--controls-️)
- [Configuration & Cache 📂](#configuration--cache-)
- [Troubleshooting & Notes 📝](#troubleshooting--notes-)

---

## Summary 📝

`spt` is a terminal UI (TUI) for interacting with Spotify. It uses:

- `spotipy` for Spotify Web API access (user & app tokens)
- `textual` for the terminal user interface
- `requests` for HTTP calls to third-party lyric providers
- (optional) `pyfiglet` for a big ASCII/FIGlet style header in the lyrics view

The script provides search (tracks, albums, artists, playlists), playback control (play/pause/next/prev/seek/volume), liked tracks management, playlist creation/import, multi-add functionality, device transfer, and lyrics display (synced when available).

---

## Features ✨

- Search tracks, albums, artists and playlists (smart search + quick prefixes)
- Play / Pause / Next / Previous / Seek / Volume / Shuffle / Repeat
- View now playing with progress bar
- Synced lyrics when available, or pseudo-synced lyrics generated from plain text
- Add tracks to playlists, create playlists, import playlists
- Multi-add / multi-review mode for batch operations
- Device management and transfer playback
- Persistent local cache and logs

---

## Requirements & Installation 🛠️

Recommended: Python 3.10+ (the script uses modern type hints and features that are easiest on 3.9/3.10+).

Suggested minimal dependencies (you can install them via pip):

```
pip install spotipy textual requests rich pyfiglet
```

If you want a fixed requirements file, the main packages are:

- spotipy
- textual
- requests
- rich (Textual depends on rich but it's good to list explicitly)
- pyfiglet (optional — used for fancy ASCII art in lyrics header)

Quick setup (Windows, PowerShell):

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install --upgrade pip
pip install spotipy textual requests rich pyfiglet
```

Then run the app:

```powershell
python "C:/route_to_script/spt_tui.py"
```

---

## Environment & Credentials 🔐

The app uses the Spotify Web API and requires credentials. Provide them either via environment variables or via the local config prompt.

Environment variables accepted:

- `SPOTIPY_CLIENT_ID` — your Spotify Client ID
- `SPOTIPY_CLIENT_SECRET` — your Spotify Client Secret
- `SPOTIPY_REDIRECT_URI` — redirect URI configured in your Spotify app (must match the one in your Spotify Developer Dashboard)

If these are not set or look like placeholders, the script will prompt you to paste credentials on first run and optionally save them to a local config file.

Scopes requested (examples used by the script):

- user-read-private
- user-read-playback-state
- user-modify-playback-state
- user-read-currently-playing
- playlist-read-private
- playlist-modify-public
- user-library-read
- user-library-modify
- user-top-read
- and more (see the script's SCOPE list)

Make sure your Spotify Developer app includes the specified redirect URI and that you grant the requested scopes when authorizing.

---

## How it works (high level) ⚙️

- The TUI is built with `textual` and mounts a left column (search / library / playlists) and a right column (details, results, lyrics, device table).
- Spotify API operations use `spotipy`. The script attempts both user-scoped requests and an app-level client when helpful (for audio analysis / features fallback).
- Lyrics are fetched via multiple strategies (LRCLib, lyrics.ovh, other heuristics). If synced lyrics (LRC) are available they are parsed and displayed with time-synced highlighting; otherwise the script can pseudo-sync plain lyrics.
- Playback synchronization: the app periodically polls Spotify playback state and updates the progress bar and lyrics highlight.
- Actions (like add to playlist, like/unlike, create playlist) are performed via the Web API and the app shows transient messages/results in the right pane.

---

## APIs & External Services 🌐

The script interacts with the following APIs/services:

- Spotify Web API (via `spotipy`) — primary for playback, search, playlists, devices and user library
- LRCLib (`https://lrclib.net`) — attempt to find synced LRC lyrics
- lyrics.ovh (`https://api.lyrics.ovh`) — fallback for plain lyrics text
- Other third-party endpoints via `requests` for lyrics lookups

Note: network requests are performed by `requests` and some calls use short timeouts by default. If an API is unavailable, the app will fall back gracefully or show an error in the log.

---

## Keybindings / Controls ⌨️

These are the primary keybindings available in the TUI (press `?` for the Help view in-app):

- Escape: Open menu / back to welcome
- Ctrl+Q: Quit
- Up / Down: Move selection
- `/`: Focus search input
- Enter: Open / Play selected item (or in Multi-Review toggles selection)
- Space: Play / Pause
- n: Next track
- p: Previous / Restart (if >3s into track, restart)
- r: Cycle repeat (off → context → track)
- c: Add selected track to Queue
- Ctrl+S: Toggle shuffle
- - / + : Volume down / up (±5%)
- m: Mute / Unmute
- Ctrl+Left / Ctrl+Right: Seek back / forward (uses configured seconds)
- <: Open seek settings
- d: Manage devices (transfer playback)
- l: Toggle Lyrics view
- Ctrl+R: Refresh
- Ctrl+T: Import playlist (paste URI)
- f: Toggle favorite (like/unlike)
- Ctrl+L: Toggle Multi-Add / Multi-Review mode
- Ctrl+A: Add to playlist (or select all in multi-review)
- Ctrl+D: Delete playlist or remove item

There are many context-sensitive actions in views (e.g., in a playlist table you can add/remove or open album/artist views). The app provides a Help view with full details.

---

## Search Quick Prefixes 🔎

You can begin a search with a prefix to limit the type:

- `/ART <query>` — search artists
- `/ALB <query>` — search albums
- `/TRK <query>` — search tracks
- `/PLY <query>` — search playlists
- `/PDC <query>` — search podcasts
- `/EPS <query>` — search episodes


If no prefix is used, a general search across types is performed.

---

## Configuration & Cache 📂

The script uses a per-user cache and log directory. By default it places data under your `Documents` folder in a subdirectory named `naarvent's projects/Spotify_TUI` (this is controlled by the `CACHE_DIR` constant in the script).

Important paths:

- Cache dir (default): `%USERPROFILE%\\Documents\\naarvent's projects\\Spotify_TUI`
- Token cache: `.cache_spotify_token` inside the cache dir
- Local config: `spt_config.json` inside the cache dir (if you choose to save credentials)
- Log file: `spt_py_textual_spotify.log` inside the cache dir

You can change environment variables or edit the script to change those defaults.

---

## Troubleshooting & Notes 📝

- If playlists don't load, try pressing `Ctrl+R` to refresh.
- If the app cannot fetch lyrics, it will try multiple providers; a fallback converts plain lyrics into pseudo-synced lines.
- For device transfer/playback actions, the target device must be available and your Spotify client must be active/online.
- The script logs detailed information to the log file in the cache dir — consult that file for debugging (`spt_py_textual_spotify.log`).
