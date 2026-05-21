# Homelab — Services, Media Pipeline & Security

**OS:** Unraid
**Container Runtime:** Docker (via Unraid Community Applications)

---

## Planned Services

| Service | Purpose | Port | Priority |
|---------|---------|------|---------|
| Jellyfin | Media server — stream ripped DVD/Blu-ray library | 8096 | #1 |
| Immich | Photo management + AI (facial recognition, duplicates) | 2283 | #2 |
| AdGuard Home | DNS-level ad blocking — runs on HP t620, NOT this server | — | Separate device |
| AutoRipper | Automated disc ripping pipeline | — | #3 |

**Note:** AdGuard intentionally kept off main server on dedicated HP t620 so DNS stays up if server reboots.

---

## Jellyfin

- **Purpose:** Serve entire ripped DVD/Blu-ray collection to all devices on network
- **Library paths:**
  - Movies: /mnt/user/media/movies/
  - TV: /mnt/user/media/tv/
- **Hardware transcoding:** Intel Arc A310 via Intel Quick Sync (QSV)
  - Codecs: AV1, HEVC 10-bit, H.264, VP9
  - Configure in Jellyfin → Dashboard → Playback → Transcoding → Intel QSV
- **Clients:** Smart TV (wired Cat6), tablets (WiFi 6E), main PC (10GbE)
- **Naming convention:**
  ```
  movies/Movie Title (Year)/Movie Title (Year).mkv
  tv/Show Name/Season 01/Show Name - S01E01 - Episode Title.mkv
  ```

---

## AutoRipper Pipeline (LG WH16NS60) — ✅ DEPLOYED 2026-05-15

- **Drive:** LG WH16NS60 — LibreDrive enabled, firmware 1.03 patched (id `4083C4CCDB14`)
- **AutoRipper:** v1.0.9 at `C:/Users/<YOU>\Desktop\AutoRipper.exe` (licensed, key screenshot in `Pictures/Screenshots/License-Keys/Autoripper Key.png`)
- **Status:** Live and ripping. First successful run: Riddick (2013) Blu-ray on 2026-05-15.

### Software Stack — Confirmed Paths

| Tool | Path | Version | Notes |
|------|------|---------|-------|
| MakeMKV | `C:\Program Files (x86)\MakeMKV\makemkvcon64.exe` | 1.18.3 | Beta key registered in `HKCU:\Software\MakeMKV\app_Key` (rotates monthly — see Maintenance below) |
| HandBrakeCLI | `C:\Program Files\HandBrake\HandBrakeCLI.exe` | 1.11.1 | Installed via winget 2026-05-15, copied to expected path |
| HandBrake GUI | `C:\Program Files\HandBrake\HandBrake.exe` | — | Only used to generate `presets.json` |
| FileBot | `C:\Program Files\FileBot\filebot.exe` | 5.2.1 | TMDB/TVDB lookup, renames + organizes |

### Storage Layout

| Stage | Location | Drive |
|-------|----------|-------|
| Temp/scratch (raw MKV + transcode workspace) | `D:\AutoRipper-Temp\` | D: drive (862GB free) |
| Final Movies | `M:\Movies\` → `\\192.168.1.10\Files\Movies` → Jellyfin lib `/mnt/user/Files/Movies` | Tower array |
| Final TV | `M:\TV\` → `\\192.168.1.10\Files\TV` → Jellyfin lib `/mnt/user/Files/TV` | Tower array |
| AutoRipper logs | `D:\AutoRipper-Temp\Logs\*.log` (one per task, named `MM.DD.YY-Title.log`) | D: drive |
| AutoRipper config | `C:/Users/<YOU>\AppData\Roaming\AutoRipper\settings.json` | — |
| AutoRipper license | `%LOCALAPPDATA%\AutoRipper\license.dat` | — |
| MakeMKV registration | `HKCU\Software\MakeMKV\app_Key` (Windows registry) | — |

### Encoder Presets

Manual select per disc type via Transcode dropdown in AutoRipper. All presets use **x265 10-bit** software encoder (CPU) for best archival quality-to-size ratio.

| Disc | Preset | CRF | Output |
|------|--------|-----|--------|
| DVD (any region) | `H.265 MKV 480p30` (best) OR `1080p30` (works, won't upscale) | 20 / 22 | Native 480p, ~1-2GB |
| Standard Blu-ray | `H.265 MKV 1080p30` | 22 | Native 1080p, ~4-8GB |
| UHD/4K Blu-ray | `H.265 MKV 2160p60 4K` | 24 | Native 2160p, ~15-25GB |
| Universal (hands-off) | `AutoRipper Universal` (custom, CRF 22, 4K cap) | 22 | Native res, slightly bigger files |

**Why CPU x265 vs NVENC:** software x265 produces ~30% smaller files at same visual quality vs NVENC HEVC. For an archival library you encode once and keep forever, CPU is the right tradeoff. RTX 2080 Ti stays free for gaming/streaming.

### Telegram Notifications

Two-layer notification system:

**Layer 1 — AutoRipper native:**
- Bot: `@your_bot_username` (token + chat ID in `settings.json`)
- Fires on: rip COMPLETE only

**Layer 2 — Sidecar notifier daemon:** `<scripts-dir>\autoripper_notifier.py`
- Watches `D:\AutoRipper-Temp\Logs\` every 15s for state transitions
- Fires on: task start, ripping, transcoding, FileBot moving, failures
- Uses the same bot token/chat ID from `settings.json`
- State persisted at `autoripper_notifier_state.json` (each log file → list of triggers already fired)
- Auto-starts at logon via Task Scheduler task `WORKSTATION-AutoRipper-Notifier`
- Bootstrap logic: on first run with empty state, marks all existing logs as fully processed (won't re-fire on stale historical logs)

### Disc Pre-Check Watcher — ✅ DEPLOYED 2026-05-16

A separate daemon that fires the moment you insert a disc (before you even open AutoRipper) and tells you whether it's worth ripping.

- **Script:** `<scripts-dir>\disc_checker.py`
- **Task Scheduler:** `WORKSTATION-Disc-Checker` (auto-starts at logon)
- **Polls:** `makemkvcon info disc:0` every 10 seconds
- **State file:** `<scripts-dir>\disc_checker_state.json`
- **Manual diagnostic:** `python disc_checker.py --once [--no-telegram]`

**How it identifies the disc:** MakeMKV's `CINFO:30` field returns the canonical English title (e.g. volume label `THE_CROODS` → title `The Croods`). No external API needed — title is matched directly against Jellyfin's SQLite DB on Tower over SSH (`/mnt/appdata/Jellyfin/data/jellyfin.db`, table `BaseItems` + `MediaStreamInfos`).

**Quality classification by main-title size:**
| Disc | Size threshold | Width compared to library |
|------|---------------|---------------------------|
| UHD  | ≥ 30 GB       | 3840 |
| BD   | 5–30 GB       | 1920 |
| DVD  | < 5 GB        | 720  |

Width (not height) is used for comparison so letterboxed films aren't misclassified (e.g. Riddick is `1920×800` because of its 2.40:1 aspect ratio — height alone would suggest DVD).

**Three verdicts pinged to Telegram:**
- 🎬 **NEW** — title not in library → ready to rip
- ✅ **HAVE** — already in library at same/better quality → eject
- ⚠️ **UPGRADE** — in library at lower quality (e.g. you have a DVD rip but this is the Blu-ray) → worth ripping for the upgrade

The same `@your_bot_username` and chat ID from `settings.json` is reused.

### End-to-End Workflow

1. Insert disc into LG WH16NS60 (drive G:)
2. Launch AutoRipper from desktop
3. Wait for drive to scan + populate `Drive` dropdown
4. Check `Transcode` checkbox, select preset matching disc type
5. Select `Movie` or `TV Show` radio
6. (Optional) Edit auto-detected title/year
7. Click `Start Ripping`
8. Title selection dialog appears → pick main title (longest runtime for movies) → Select
9. Pipeline runs:
   - MakeMKV rips raw MKV to `D:\AutoRipper-Temp\<Title>_<Year>\` (~25-35 GB for Blu-ray, ~20-40 min)
   - HandBrake transcodes raw → final H.265 MKV in temp folder (CPU-bound, 30-90 min for Blu-ray)
   - FileBot queries TMDB/TVDB, renames to convention pattern
   - File moves to `M:\Movies\<Movie Name (Year)>\<Movie Name (Year)>.mkv` over 10GbE
10. Jellyfin library auto-scan picks up the new file (1-5 min)

### FileBot Naming Conventions (from settings.json)

```
Movies:  {n} ({y})/{n} ({y})           → "Riddick (2013)/Riddick (2013).mkv"
TV:      {n}/Season {s}/{n} - {s00e00} - {t}  → "Show Name/Season 1/Show Name - 1x01 - Pilot.mkv"
```

### Maintenance

- **MakeMKV beta key rotates ~monthly.** When MakeMKV throws shareware prompt again (typically every 30 days), fetch new key from https://forum.makemkv.com/forum/viewtopic.php?t=1053 and update registry: `Set-ItemProperty -Path 'HKCU:\Software\MakeMKV' -Name 'app_Key' -Value '<new key>'`
- **HandBrake presets:** stored at `%APPDATA%\HandBrake\presets.json`. Custom `AutoRipper Universal` preset injected here. Backup at `presets.json.bak.<timestamp>`.
- **Temp folder:** auto-clears per-rip when AutoRipper finishes successfully. If a rip fails, leftover files in `D:\AutoRipper-Temp\<Title>\` need manual cleanup.
- **Telegram bot:** token in AutoRipper `settings.json` AND used by sidecar notifier. Single source of truth.

---

## Immich — Photo Management

- **Purpose:** Self-hosted Google Photos replacement with AI features
- **Library path:** /mnt/user/photos/
- **Machine learning:** Run on CPU (Arc A310 reserved for Jellyfin)
  - Initial library scan: Run overnight — CPU handles it fine
  - Ongoing: Only processes new photos (minimal CPU load)

### AI Features
- Facial recognition — group by person automatically
- Duplicate detection — finds duplicates even with different filenames
- Object/scene detection — tag photos by content
- Date/location organization — uses EXIF metadata
- RAW file support — handles Nikon P950 RAW files

### Migration Plan (Day 1)
1. Copy all Samsung T7 content (family photos/docs) to server
2. Point Immich at /mnt/user/photos/
3. Let initial ML scan run overnight
4. Review facial recognition groups and tag people
5. Set up mobile auto-backup from phone to Immich

### Camera Sources
| Camera | Format | Resolution |
|--------|--------|------------|
| DJI Air 3 | MP4 / JPEG | 4K video, 48MP photos |
| Insta360 X3 | INSV / JPEG | 5.7K 360° |
| GoPro Hero 5 | MP4 / JPEG | 4K video |
| Nikon COOLPIX P950 | NRW (RAW) / JPEG | 16MP |
| Phone | HEIC / JPEG | [varies] |

---

## Network Security

### Local Access
- Unraid web UI on local network only (no direct internet exposure recommended)
- All Docker services accessible on LAN

### Remote Access Plan
- **Recommended:** Tailscale VPN
  - Install Tailscale on Unraid (Community Applications)
  - Install on phone, main PC, any device needing remote access
  - No port forwarding required — zero trust mesh VPN
  - Free tier supports personal use

### DNS
- **Primary DNS:** HP t620 Plus running AdGuard Home (replacement unit — original t620 dead, won't power on; t620 Plus en route)
- **Backup DNS:** HP HSTNC-006-TC (dns2) — Alpine Linux 3.23.3 x86, dnsmasq forwarding to t620 AdGuard, fallback 8.8.8.8. Static IP: 192.168.1.5. Status: COMPLETE and running.
- **Router config:** Primary DNS → t620 Plus IP, Secondary DNS → 192.168.1.5 (dns2)
- **Result:** DNS infrastructure stays up independent of main server

### General Security Notes
- Do NOT expose Jellyfin or Immich directly to internet without reverse proxy + auth
- If exposing any service externally: Nginx Proxy Manager + Let's Encrypt SSL + Authelia 2FA
- SSH key auth only — disable password SSH once server is stable
- Keep Unraid and all Docker containers updated

---

## HP t620 Plus — AdGuard Home + WireGuard + Uptime Kuma

- **Status:** Replacement unit — original t620 dead (won't power on, salvaged for parts). t620 Plus en route.
- **OS:** Debian or Ubuntu Server (minimal)
- **Services:** AdGuard Home + WireGuard VPN + Uptime Kuma (all three consolidated)
- **Power draw:** ~5-8W continuous — very cheap to run 24/7
- **AdGuard install:** `curl -s -S -L https://raw.githubusercontent.com/AdguardTeam/AdGuardHome/master/scripts/install.sh | sh`
- **AdGuard web UI port:** 3000 (setup), 80 (after setup)
- **DNS port:** 53
- **Upstream DNS:** 1.1.1.1 + 8.8.8.8
- **WireGuard port:** UDP 51820 (port forward on router)
- **Uptime Kuma port:** 3001

## HP HSTNC-006-TC — dns2 (Backup DNS)

- **Status:** COMPLETE and operational
- **Role:** Backup DNS resolver (dns2) — dnsmasq forwarding to t620 AdGuard, fallback 8.8.8.8
- **OS:** Alpine Linux Extended 3.23.3 x86 (32-bit — only viable OS for Atom N280)
- **Static IP:** 192.168.1.5 (DHCP reservation in router)
- **Note:** Hardware is dead / being used for parts. dns2 role will need to be migrated if unit fully fails.
- **Reference:** `dns2-alpine-setup.md`

---

## Future Services (When Ready)

| Service | Purpose | Notes |
|---------|---------|-------|
| Tailscale | Remote VPN access | Recommended first add-on |
| Nginx Proxy Manager | Reverse proxy + SSL | If exposing any service externally |
| Uptime Kuma | Service monitoring + alerts | Lightweight, runs in Docker |
| Vaultwarden | Self-hosted password manager | Optional — Bitwarden compatible |
| Sonarr / Radarr | Automated TV/Movie management | Pairs with Jellyfin |

---

## Open Tasks

- [x] ~~Finish 3D printing case and assemble server~~ — COMPLETE
- [ ] Install Unraid — configure array and cache
- [ ] Day 1: Migrate Samsung T7 to protected array
- [ ] Install Jellyfin container — configure Arc A310 QSV transcoding
- [ ] Install Immich container — point at photos share, run initial scan
- [ ] Set up AutoRipper pipeline (rip first batch of discs)
- [ ] Set up HP t620 Plus with AdGuard Home + WireGuard + Uptime Kuma (unit en route)
- [x] ~~Set up HSTNC as backup DNS~~ — COMPLETE: dns2 running Alpine dnsmasq at 192.168.1.5
- [ ] Install Tailscale on server and main PC
- [ ] Configure router DNS to point at both thin clients
