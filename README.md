# 🎵 The Ultimate Navidrome Setup Guide

**A complete, beginner-friendly guide to getting the very best out of your self-hosted Navidrome music server — from installation to perfect metadata, playback, and beyond.**

*Based on community recommendations and best practices.*

<a href="https://buymeacoffee.com/succinctrecords"><img src="https://img.shields.io/badge/Buy%20Me%20A%20Coffee-Support-yellow?logo=buy-me-a-coffee"></a>

---

## 📋 Table of Contents

1. [Overview — What Are We Building?](#-overview--what-are-we-building)
2. [Step 1: Install Navidrome (Docker)](#-step-1-install-navidrome-docker)
3. [Step 2: Tag Your Music with MusicBrainz Picard](#-step-2-tag-your-music-with-musicbrainz-picard)
4. [Step 3: Fix Genre Tags](#-step-3-fix-genre-tags)
5. [Step 4: Set Up Playback Apps](#-step-4-set-up-playback-apps)
6. [Step 5: Remote Access with Tailscale](#-step-5-remote-access-with-tailscale)
7. [Step 6: Add Radio Stations](#-step-6-add-radio-stations)
8. [Step 7: View Your Listening Stats (Navidrome Wrapped)](#-step-7-view-your-listening-stats-navidrome-wrapped)
9. [Audio Quality Tips](#-audio-quality-tips)
10. [Alternatives Considered](#-alternatives-considered)
11. [Quick Reference Links](#-quick-reference-links)

---

## 🗺️ Overview — What Are We Building?

Here's the full stack at a glance:

| Component | Purpose | Link |
|-----------|---------|------|
| **Navidrome** | Self-hosted music server | [navidrome.org](https://www.navidrome.org/) |
| **MusicBrainz Picard** | Auto-tag metadata (artist, album, track, etc.) | [picard.musicbrainz.org](https://picard.musicbrainz.org/) |
| **Picard Filenaming Script Generator** | Generate Picard file-naming scripts without coding | [GitHub](https://github.com/WB2024/WBs-Picard-Filenaming-Script-Generator) |
| **Essentia Music Tagger** | AI-powered genre/mood tagging (replaces Picard's genres) | [GitHub](https://github.com/WB2024/Essentia-to-Metadata) |
| **Artist Genre Metadata Enforcer** | Define your own genre per artist, enforce it everywhere | [GitHub](https://github.com/WB2024/Artist-Genre-Metadata-Enforcer) |
| **Navidrome Radio Station Manager** | Add internet radio stations to Navidrome | [GitHub](https://github.com/WB2024/Add-Navidrome-Radios) |
| **Navidrome Wrapped** | Spotify Wrapped–style listening stats | [GitHub](https://github.com/mdeik/Navidrome-Wrapped) |
| **Feishin** | Desktop music player (Spotify-like UI) | [GitHub](https://github.com/jeffvli/feishin) |
| **Symfonium** | Mobile music player (Android) | [symfonium.app](https://symfonium.app/) |
| **Tailscale** | Secure remote access (no port forwarding) | [tailscale.com](https://tailscale.com/) |

```
┌─────────────┐     ┌──────────────┐     ┌──────────────┐
│  Your Music  │────▶│   Picard     │────▶│  Genre Fix   │
│  (files)     │     │  (tagging)   │     │  (Essentia   │
│              │     │              │     │   or Manual)  │
└─────────────┘     └──────────────┘     └──────┬───────┘
                                                 │
                                                 ▼
┌─────────────┐     ┌──────────────┐     ┌──────────────┐
│  Feishin    │◀────│  Navidrome   │◀────│  Clean Music  │
│  (desktop)  │     │  (server)    │     │  Library      │
├─────────────┤     │              │     └──────────────┘
│  Symfonium  │◀────│              │
│  (mobile)   │     └──────┬───────┘
└─────────────┘            │
                    ┌──────┴───────┐
                    │  Tailscale   │
                    │  (remote)    │
                    └──────────────┘
```

---

## 🐳 Step 1: Install Navidrome (Docker)

### Prerequisites

- Docker and Docker Compose installed on your machine
- A music library folder on your system
- (Optional) [Spotify](https://developer.spotify.com/) and [Last.fm](https://www.last.fm/api) API keys for artist images and scrobbling

### Docker Compose Configuration

Create a `docker-compose.yml` file:

```yaml
services:
  navidrome:
    image: ghcr.io/navidrome/navidrome:latest
    container_name: navidrome
    environment:
      - ND_MUSICFOLDER=/music
      - ND_PORT=4533
      - ND_SPOTIFY_ID=your_spotify_client_id        # For artist images
      - ND_SPOTIFY_SECRET=your_spotify_client_secret  # For artist images
      - ND_LASTFM_APIKEY=your_lastfm_api_key         # For scrobbling
      - ND_LASTFM_SECRET=your_lastfm_secret           # For scrobbling
      - ND_LOGLEVEL=info
    volumes:
      - /path/to/your/navidrome/data:/data       # Navidrome database & config
      - /path/to/your/music/library:/music:ro    # Your music (read-only)
    ports:
      - "4533:4533"
    restart: unless-stopped
```

### Getting Your API Keys

#### Spotify (for artist images & artwork)
1. Go to [developer.spotify.com](https://developer.spotify.com/)
2. Log in and create a new app
3. Copy the **Client ID** and **Client Secret**
4. Paste them into `ND_SPOTIFY_ID` and `ND_SPOTIFY_SECRET`

#### Last.fm (for scrobbling)
1. Go to [last.fm/api](https://www.last.fm/api)
2. Create a new API account/application
3. Copy the **API Key** and **Shared Secret**
4. Paste them into `ND_LASTFM_APIKEY` and `ND_LASTFM_SECRET`

### Launch Navidrome

```bash
docker compose up -d
```

Visit `http://your-server-ip:4533` in your browser. On first launch, you'll create an admin account.

> 💡 **Tip:** Mount your music folder as `:ro` (read-only) so Navidrome can't accidentally modify your files.

---

## 🏷️ Step 2: Tag Your Music with MusicBrainz Picard

**MusicBrainz Picard** is the go-to tool for automatically tagging your music with accurate metadata (artist, album, title, track number, cover art, etc.). It looks up your music against the massive [MusicBrainz](https://musicbrainz.org/) database — which has excellent coverage for Japanese and Korean music too.

### Option A: Run Picard via Docker (Recommended for Servers)

If your music is on a headless server, run Picard as a web-accessible Docker container:

```yaml
services:
  picard-web:
    image: aandree5/picard-web:latest
    container_name: picard-web
    restart: unless-stopped
    ports:
      - "8086:5000"
    volumes:
      - /path/to/picard/config:/picard-web:rw    # Picard config/state
      - /path/to/your/music:/music:rw            # Your music (read-write!)
```

```bash
docker compose up -d
```

Access Picard in your browser at `http://your-server-ip:8086`.

> ⚠️ **Important:** The music volume must be `:rw` (read-write) here because Picard needs to write tags to your files.

### Option B: Run Picard on Desktop

Download from [picard.musicbrainz.org](https://picard.musicbrainz.org/) and install normally on Windows, macOS, or Linux.

### How to Use Picard (Quick Workflow)

1. **Add files** — Drag & drop your music folder into Picard
2. **Cluster** — Click "Cluster" to group files by album
3. **Lookup** — Click "Lookup" to match against MusicBrainz
4. **Review** — Check the matches are correct (green = good match)
5. **Save** — Click "Save" to write the tags to your files

### Generate a File Naming Script

Picard can also **rename and reorganize** your files based on metadata. Writing these scripts manually is complex, so use the **Picard Filenaming Script Generator**:

📦 **[WB's Picard Filenaming Script Generator](https://github.com/WB2024/WBs-Picard-Filenaming-Script-Generator)**

This interactive CLI tool lets you:
- Choose from **6 ready-made presets** (e.g., `Artist/[Year] Album/01 - Track.flac`)
- Or use the **custom builder wizard** to configure every detail
- Preview example output before saving
- Export directly into Picard — no scripting knowledge needed

```bash
# Install and run
git clone https://github.com/WB2024/WBs-Picard-Filenaming-Script-Generator.git
cd WBs-Picard-Filenaming-Script-Generator
python3 picard_script_generator.py
```

> 💡 **For Japanese/Korean music:** MusicBrainz has strong coverage of J-Pop, K-Pop, anime OSTs, and more. Picard will pull the correct metadata including original-language titles when available.

---

## 🎸 Step 3: Fix Genre Tags

Picard is great for most metadata, but its **genre tags are often generic or inconsistent** (e.g., tagging everything as "Rock" or "Pop"). There are two approaches to fix this:

### Option A: AI-Powered Genre Tagging with Essentia (Automatic)

📦 **[Essentia Music Tagger](https://github.com/WB2024/Essentia-to-Metadata)**

This tool **analyzes the actual audio waveform** using machine learning to predict accurate genres and moods. No internet required after setup — everything runs locally.

**What it does:**
- Classifies across **400 genre categories** (Discogs taxonomy)
- Detects **moods** (energetic, dark, happy, aggressive, etc.)
- Writes `GENRE` and `MOOD` tags directly to your files
- Supports FLAC & MP3

**Quick start:**

```bash
# Clone and set up
git clone https://github.com/WB2024/Essentia-to-Metadata.git
cd Essentia-to-Metadata

# Create virtual environment
python3 -m venv venv
source venv/bin/activate

# Install dependencies
pip install essentia-tensorflow mutagen numpy

# Download ML models (~87MB)
bash download_models.sh

# Run (always do a dry run first!)
python tag_music.py
```

**Or via Docker:**

```bash
# CPU mode
docker compose build essentia-tagger
MUSIC_DIR=/path/to/music docker compose run --rm essentia-tagger

# Dry run first!
MUSIC_DIR=/path/to/music docker compose run --rm essentia-tagger /music --auto --dry-run
```

**Example output:**

```
Input:  2Pac - Temptations.flac
Output: GENRE = Hip-Hop - Gangsta; Hip-Hop - East Coast Hip Hop
        MOOD  = Energetic; Dark
```

> 🧪 **Always run in dry-run mode first** to preview what changes will be made!

---

### Option B: Manual Genre Enforcement (Your Rules)

📦 **[Artist Genre Metadata Enforcer](https://github.com/WB2024/Artist-Genre-Metadata-Enforcer)**

If you prefer **your own genre definitions** rather than what an algorithm or database decides, this tool lets you define a genre once per artist and apply it to their entire discography.

**Why use this?**
- You decide that all 2Pac is `Hip-Hop`, period
- No more "Hip Hop" vs "Hip-Hop" vs "Rap" inconsistencies
- Define once, enforce everywhere — even across thousands of tracks

**Quick start:**

```bash
# Install dependency
sudo apt install python3-mutagen   # Debian/Ubuntu
# or: pip install mutagen

# Download and install
sudo curl -o /usr/local/bin/music-genre-enforcer \
  https://raw.githubusercontent.com/WB2024/Artist-Genre-Metadata-Enforcer/main/music-genre-enforcer.py
sudo chmod +x /usr/local/bin/music-genre-enforcer

# Run
music-genre-enforcer
```

**Interactive workflow:**

```
Genre for [Aphex Twin] (or l/p/?/s): p
Existing genres:
  [1] Electronic - Ambient
  [2] Hip-Hop
  [3] Post-Punk
  [4] Rock - Art
Pick number (blank cancels): 1
  -> Set [Aphex Twin] = Electronic - Ambient
```

**Expected folder structure:**

```
/your/music/library/
├── A/
│   ├── Aphex Twin/
│   │   ├── [1992] Selected Ambient Works 85-92/
│   │   │   ├── 01 - Xtal.flac
├── B/
│   ├── Bowie, David/
├── #/
│   └── 2Pac/
└── Various Artists/
```

| Feature | Essentia (Option A) | Genre Enforcer (Option B) |
|---------|-------------------|-------------------------|
| Automation | Fully automatic (AI) | Semi-automatic (you define, it applies) |
| Accuracy | Based on audio analysis | Based on your personal preference |
| Best for | Large libraries, discovery | Curated libraries, consistency |
| Multi-genre | Yes (configurable) | Single genre per artist |

> 💡 **Pro tip:** You can use **both** — run Essentia first for a baseline, then use the Genre Enforcer to override specific artists with your preferred labels.

---

## 🎧 Step 4: Set Up Playback Apps

### Desktop: Feishin

📦 **[Feishin](https://github.com/jeffvli/feishin)** — A modern, Spotify-like desktop music player.

**Why Feishin?**
- Beautiful UI that looks and feels like Spotify
- Connects directly to Navidrome (and Jellyfin)
- Dual playback engines: **MPV** (high-quality native) or **Web player**
- Smart playlists, lyrics, scrobbling, themes
- Available on **Windows, macOS, and Linux**

**Download:** [github.com/jeffvli/feishin/releases](https://github.com/jeffvli/feishin/releases)

**Setup:**
1. Download and install Feishin for your OS
2. Open Feishin → Add Server → Enter your Navidrome URL and credentials
3. Start browsing and playing!

#### 🔊 Optimal Audio Setup (Advanced)

For the best audio quality on desktop:

1. **Use MPV as the playback engine** in Feishin's settings
2. **Connect a USB DAC** (Digital-to-Analog Converter) to your computer
3. **Set audio output to WASAPI** (Windows) in Feishin/MPV settings
4. **Enable audio-exclusive mode** — this prevents Windows from resampling your audio

> This bypasses the Windows audio mixer entirely, sending bit-perfect audio to your DAC.

---

### Mobile: Symfonium

📦 **[Symfonium](https://symfonium.app/)** — The best Android music player for self-hosted servers.

**Why Symfonium?**
- Connects to Navidrome (plus Plex, Jellyfin, Emby, cloud storage, and more)
- Offline caching — download music for when you're without internet
- Gapless playback, replay gain, and high-res audio support
- Cast to Chromecast, DLNA, Sonos, and more
- Android Auto & Wear OS support
- One-time purchase, no subscriptions, no ads

**Setup:**
1. Install from [Google Play Store](https://play.google.com/store/apps/details?id=app.symfonik.music.player)
2. Add your Navidrome server (URL + credentials)
3. Enjoy your music on the go!

---

## 🌐 Step 5: Remote Access with Tailscale

Want to listen to your music when you're away from home? **Don't expose your server to the internet** with port forwarding — use **Tailscale** instead.

📦 **[Tailscale](https://tailscale.com/)** — Creates a secure, private network (VPN) between your devices with zero configuration.

### How It Works

1. Install Tailscale on your **server** (where Navidrome runs)
2. Install Tailscale on your **phone/laptop** (where you listen)
3. Both devices get a private Tailscale IP address
4. Access Navidrome using the Tailscale IP — encrypted, secure, no port forwarding

### Setup

```bash
# On your server (Debian/Ubuntu)
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up

# Note the Tailscale IP it gives you (e.g., 100.x.y.z)
```

Install the Tailscale app on your phone/laptop from [tailscale.com/download](https://tailscale.com/download).

Then in Feishin or Symfonium, set your server URL to:
```
http://100.x.y.z:4533
```

> 🔒 **Why Tailscale?** No ports to open, no dynamic DNS to configure, encrypted WireGuard tunnel, and a generous free tier for personal use.

---

## 📻 Step 6: Add Radio Stations

📦 **[Navidrome Radio Station Manager](https://github.com/WB2024/Add-Navidrome-Radios)**

A CLI tool to search, browse, and add internet radio stations to your Navidrome server from the [Radio-Browser](https://www.radio-browser.info/) community database.

### Install

```bash
# Install dependency
pip3 install requests

# Download and install
curl -o /usr/local/bin/navidrome-radio \
  https://raw.githubusercontent.com/WB2024/Add-Navidrome-Radios/main/navidrome-radio.py
sudo chmod +x /usr/local/bin/navidrome-radio

# Run
navidrome-radio
```

### Usage

On first run, enter your Navidrome database path (e.g., `/path/to/navidrome/data/navidrome.db`).

Then search and add stations:

```
MAIN MENU
1. Search and add radio stations
2. View existing stations in database
3. Inspect database (debug)
4. Change database path
5. Exit

Search by name, genre, country, or browse top-voted stations.
Select stations → type "add" → they appear in Navidrome immediately!
```

> 💡 **Docker users:** Your database path is on the **host**, not inside the container. Check your `docker-compose.yml` volumes to find it.

---

## 📊 Step 7: View Your Listening Stats (Navidrome Wrapped)

📦 **[Navidrome Wrapped](https://github.com/mdeik/Navidrome-Wrapped)**

**Navidrome Wrapped** is a beautiful, Spotify Wrapped–inspired client-side web app that generates comprehensive **all-time listening statistics** from your Navidrome server. See your top artists, albums, songs, genre and decade breakdowns, diversity scoring, rating analysis, audio quality insights, and shareable summary cards — all in a stunning interactive slideshow.

### Key Features

- **Pure client-side** — no backend server needed; runs entirely in your browser
- **Privacy-first** — your credentials are never stored or sent anywhere except directly to your Navidrome instance; all processing is local
- **Interactive slideshow** with 8 sections: Welcome, Listening Time, Top Artists, Top Albums, Top Songs, Diversity, Ratings, and Summary
- **Interactive charts** for genre and decade distribution
- **Shareable cards** — generates 9:16 share images of your stats
- **Demo mode** — try it with sample data before connecting your server
- **Caches results** in localStorage for 90 days so repeat visits are instant
- **Live hosted version** available at [mdeik.github.io/Navidrome-Wrapped](https://mdeik.github.io/Navidrome-Wrapped/)

### How to Use

#### Option A: Use the Hosted Version (Easiest)

Just visit **[mdeik.github.io/Navidrome-Wrapped](https://mdeik.github.io/Navidrome-Wrapped/)** and enter:
- Your Navidrome server URL (e.g., `http://your-server-ip:4533` or `https://your-domain.com`)
- Your username and password
- Click **"Generate All-Time Wrapped"**

#### Option B: Self-Host It

It's just static files — no build step needed:

```bash
git clone https://github.com/mdeik/Navidrome-Wrapped.git
cd Navidrome-Wrapped
# Open index.html in your browser, or serve it:
python3 -m http.server 8080
# Then visit http://localhost:8080
```

> 💡 **Tailscale users:** If accessing your Navidrome remotely, use your Tailscale IP as the server URL (e.g., `http://100.x.y.z:4533`).

> 🎮 **Try demo mode first** — Navidrome Wrapped includes a demo mode with sample data so you can explore the interface before connecting your own server.

---

## 🎶 Audio Quality Tips

These tips will help you get the best listening experience:

### Building a Clean Library

| Recommendation | Details |
|----------------|---------|
| **Audio format** | FLAC (lossless) is ideal. Aim for **16-bit / 44.1kHz** minimum |
| **Single format** | Keep everything in one format for consistency (e.g., all FLAC) |
| **Storage** | If storage is limited and quality isn't a priority, MP3 is fine |
| **Organization** | Use Picard to rename/organize files into a consistent folder structure |

### Hardware Recommendations

| Component | Recommendation |
|-----------|---------------|
| **DAC** | Get a dedicated USB DAC instead of using your motherboard's audio |
| **Speakers/Headphones** | Invest in quality — this is where you'll hear the biggest difference |
| **Amplifier** | Match your amp to your speakers/headphones |

### Software Audio Chain (Desktop)

```
Feishin → MPV Engine → WASAPI (Exclusive) → USB DAC → Amp → Speakers
```

This chain ensures:
- ✅ Bit-perfect audio output
- ✅ No Windows audio mixer resampling
- ✅ Direct hardware access
- ✅ Maximum quality from your files

---

## 🔄 Alternatives Considered

These alternatives are worth knowing about:

| Tool | Type | Verdict |
|------|------|---------|
| **Lidarr** | Automated music library manager (like Sonarr/Radarr) | Divisive — great for other *arr apps, but can be frustrating for music. Not recommended for beginners |
| **beets** | CLI music tagger & library manager | Excellent and powerful, but CLI-only. If you're comfortable with terminals, it's a great choice. GUIs exist but aren't very stable |
| **MusicBrainz Picard** | GUI music tagger | **Recommended** — user-friendly GUI, great database coverage, works well for all languages including Japanese & Korean |

---

## 🔗 Quick Reference Links

### Core Infrastructure
| Tool | Link |
|------|------|
| Navidrome | [navidrome.org](https://www.navidrome.org/) |
| MusicBrainz Picard | [picard.musicbrainz.org](https://picard.musicbrainz.org/) |
| Feishin (Desktop Player) | [github.com/jeffvli/feishin](https://github.com/jeffvli/feishin) |
| Symfonium (Mobile Player) | [symfonium.app](https://symfonium.app/) |
| Tailscale (Remote Access) | [tailscale.com](https://tailscale.com/) |

### Enhancement Tools
| Tool | Link |
|------|------|
| Picard Filenaming Script Generator | [github.com/WB2024/WBs-Picard-Filenaming-Script-Generator](https://github.com/WB2024/WBs-Picard-Filenaming-Script-Generator) |
| Essentia Music Tagger (AI Genres) | [github.com/WB2024/Essentia-to-Metadata](https://github.com/WB2024/Essentia-to-Metadata) |
| Artist Genre Metadata Enforcer | [github.com/WB2024/Artist-Genre-Metadata-Enforcer](https://github.com/WB2024/Artist-Genre-Metadata-Enforcer) |
| Navidrome Radio Station Manager | [github.com/WB2024/Add-Navidrome-Radios](https://github.com/WB2024/Add-Navidrome-Radios) |
| Navidrome Wrapped (Listening Stats) | [github.com/mdeik/Navidrome-Wrapped](https://github.com/mdeik/Navidrome-Wrapped) |

---

## ✅ Suggested Order of Operations

Here's the recommended sequence for setting everything up from scratch:

```
1. 🐳  Deploy Navidrome via Docker
2. 🏷️  Tag your music with Picard (get metadata right first)
3. 📁  Generate a filenaming script to organize your library
4. 🎸  Fix genres with Essentia and/or Genre Enforcer
5. 🎧  Install Feishin (desktop) and/or Symfonium (mobile)
6. 🌐  Set up Tailscale for remote access
7. 📻  Add radio stations
8. 📊  View your listening stats with Navidrome Wrapped
9. 🔊  Optimize your audio hardware chain
10. 🎶  Enjoy your music!
```

---

*Happy listening!* 🎶

---

## ☕ Support

If you found this guide helpful, consider buying me a coffee!

<a href="https://buymeacoffee.com/succinctrecords"><img src="https://img.shields.io/badge/Buy%20Me%20A%20Coffee-Support-yellow?logo=buy-me-a-coffee"></a>
