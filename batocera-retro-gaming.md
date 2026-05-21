# Batocera Setup — Retro Gaming Rig

**Hardware:** i7-4770 / Asus B85M-E / GTX 1650 / 32GB DDR3
**Status:** Operational
**Use case:** Local play + Moonlight game streaming via Sunshine
**IP:** 192.168.1.187 | **SSH:** root / linux

---

## Storage Layout

| Drive | Size | Role |
|-------|------|------|
| SanDisk USB | 30GB | Batocera OS + BIOS files + saves + settings |
| Samsung USB 3.1 #1 | 256GB | PS2 ROMs |
| Samsung USB 3.1 #2 | 256GB | GameCube ROMs |
| Samsung USB 3.1 #3 | 256GB | Everything else (PS1, N64, SNES, etc.) |

**Label all Samsung drives** before first use — Batocera mounts by label at `/media/LABEL`.
Recommended labels: `ROMS-PS2`, `ROMS-GC`, `ROMS-OTHER`

---

## Step 1 — Flash Batocera to SanDisk

1. Download **Batocera x86_64** from [batocera.linux.com](https://batocera.linux.com/upgrade/x86_64/stable/last/) — get the `.img.gz`
2. Flash with **Rufus** on Windows:
   - Device: SanDisk 30GB
   - Boot selection: batocera `.img.gz` (Rufus handles gz directly)
   - Partition scheme: **GPT**
   - Write mode: **DD Image** (Rufus will prompt — select DD)
3. Batocera auto-expands the userdata partition to fill the drive on first boot

---

## Step 2 — First Boot

Boot order in BIOS: USB first. Batocera will expand partitions and land on EmulationStation.

### Set Nvidia Driver (critical — do this first)
`Main Menu → System Settings → Hardware → GPU Driver → nvidia`

Reboot when prompted. Without this, you're on nouveau — poor performance and no Vulkan.

### Network
`Main Menu → Network Settings → Enable WiFi` (or plug LAN — auto-detected)

### Enable SSH
`Main Menu → System Settings → Security → Enable SSH`
- Default credentials: `root` / `linux`
- Change password immediately: `passwd` via SSH

---

## Step 3 — Mount ROM Drives

Label the Samsung drives on a Windows machine first:
```
# PowerShell — run as admin, adjust drive letters
Set-Volume -DriveLetter E -NewFileSystemLabel "ROMS-PS2"
Set-Volume -DriveLetter F -NewFileSystemLabel "ROMS-GC"
Set-Volume -DriveLetter G -NewFileSystemLabel "ROMS-OTHER"
```

Plug all three into the rig. Batocera auto-mounts them:
- `/media/ROMS-PS2`
- `/media/ROMS-GC`
- `/media/ROMS-OTHER`

### Configure ROM Paths (SSH or F1 → File Manager)

Edit `/userdata/system/batocera.conf` and add:

```ini
# ROM paths — external USB drives
global.roms=/userdata/roms
pcsx2.rompath=/media/ROMS-PS2
dolphin.rompath=/media/ROMS-GC
```

Or use symlinks (cleaner):
```bash
rm -rf /userdata/roms/ps2 && ln -s /media/ROMS-PS2 /userdata/roms/ps2
rm -rf /userdata/roms/gamecube && ln -s /media/ROMS-GC /userdata/roms/gamecube
rm -rf /userdata/roms/wii && ln -s /media/ROMS-GC/wii /userdata/roms/wii
```

Symlinks persist across reboots because `/userdata` is on the persistent partition.

---

## Step 4 — BIOS Files

Place in `/userdata/bios/`

| System | Required File | Notes |
|--------|--------------|-------|
| PS2 | `ps2-0230a-20080220.bin` | Required — place in `/userdata/bios/ps2/` subfolder. MD5: `21038400dc633070a78ad53090c53017` |
| PS1 | `scph5501.bin` | Required — place in `/userdata/bios/`. MD5: `490f666e1afb15b7362b406ed1cea246` |
| GameCube | `GC/USA/IPL.bin` | Optional but suppresses warning. MD5: `b17148254a5799684c7d783206504926` |
| Dreamcast | `dc_boot.bin`, `dc_flash.bin` | Required |
| Saturn | `saturn_bios.bin` | Required |

Verify BIOS detection: `Main Menu → Game Settings → Missing BIOS`

---

## Step 5 — Emulator Config

### PS2 (PCSX2-QT)
- Batocera ships PCSX2-QT as default for PS2
- GPU: set to **Vulkan** (GTX 1650 supports it, best performance)
- Resolution: **2x native** (2x is 720p → fine for 1080p display, 3x is 1080p, 4x may stutter on some titles)
- Per-game overrides available via holding Start on a game

### GameCube / Wii (Dolphin)
- Backend: **Vulkan**
- Enable **Dual Core** (default on) — i7-4770 handles it
- Internal resolution: **2x (1080p)** — push to 3x for less demanding games
- Wii ROMs go in `/userdata/roms/wii` — Dolphin handles both from same emulator

### N64 (Mupen64Plus-Next via RetroArch)
- Default core is fine
- Use **GlideN64** video plugin (default)

### Performance expectations on i7-4770 + GTX 1650

| System | Status |
|--------|--------|
| PS1, N64, SNES, GBA | Trivial — full speed |
| PS2 | Excellent — nearly everything full speed |
| GameCube / Wii | Very good — most games full speed |
| Dreamcast | Full speed |
| Xbox OG (Xemu) | Good — most titles playable |
| PS3 (RPCS3) | Hit or miss — limited by i7-4770 single-core IPC |
| Switch (Yuzu/Ryujinx) | Borderline — older/simpler titles only |

---

## Step 6 — Scraper (Metadata + Box Art)

`Main Menu → Scraper`
- Source: **ScreenScraper** (create free account at screenscraper.fr)
- Scrape: box art, description, video snaps (optional — use disk space)
- Scrape by system, not all at once — PS2 library will take a while

---

## Step 7 — Bluetooth Controller Setup

`Main Menu → Controller Settings → Pair a Bluetooth Controller`

Batocera has a built-in Bluetooth pairing UI — no need to touch the CLI.

**Common controllers:**
- **Xbox Series controller:** Hold pair button (top of controller) until light flashes → select in Batocera
- **PS5 DualSense:** Hold PS + Create until light bar flashes → select in Batocera
- **8BitDo:** Hold Start+B for 2 seconds to enter pairing mode → select in Batocera
  - Use **XInput mode** (Start+X on power-on) for best compatibility in Batocera

Map buttons when prompted. Batocera stores per-controller profiles — configure once, persists across reboots.

---

## Step 8 — Sunshine Streaming Setup

Sunshine runs on the Batocera machine; Moonlight runs on the client (phone, PC, TV stick, etc.).

### Install Sunshine on Batocera

Sunshine is installed as a **Flatpak** (not AppImage). Batocera's built-in Flatpak support handles this.

Config location: `/userdata/saves/flatpak/data/.var/app/dev.lizardbyte.app.Sunshine/config/sunshine/sunshine.conf`

### Sunshine Config (sunshine.conf)

Locked down for headless streaming — prevents resolution negotiation issues:

```ini
# Sunshine config - resolution locked to prevent blurry reconnects
resolutions = [
    1920x1080
]
fps = [30, 60]
encoder = nvenc
min_log_level = info
```

### Autostart

Batocera runs `/userdata/system/custom.sh` on boot. Current contents:

```bash
#!/bin/bash
# Force 1080p and disable panning for headless streaming
(sleep 8 && DISPLAY=:0 xrandr --output HDMI-0 --mode 1920x1080 --rate 60 --panning 0x0 2>/dev/null) &
```

Sunshine itself auto-starts via Flatpak/systemd — no manual launch needed in custom.sh.

### Headless Display Fix (Critical)

Without a monitor connected, the GTX 1650 reports all outputs as "disconnected" and Xorg defaults to an 8x8 virtual screen. This causes Moonlight to display a tiny/blurry image.

**Fix applied:** Custom xorg config + EDID file that tricks the GPU into thinking a 1080p monitor is connected.

**File 1:** `/etc/X11/xorg.conf.d/10-headless-display.conf`
```
Section "Device"
    Identifier     "GPU0"
    Driver         "nvidia"
    Option         "ConnectedMonitor" "HDMI-0"
    Option         "CustomEDID" "HDMI-0:/etc/X11/edid-1080p.bin"
    Option         "AllowExternalGpus" "True"
    Option         "ModeValidation" "NoMaxPClkCheck, NoEdidMaxPClkCheck, NoMaxSizeCheck, NoHorizSyncCheck, NoVertRefreshCheck, NoDualLinkDVICheck, NoDisplayPortBandwidthCheck"
EndSection

Section "Screen"
    Identifier     "Screen0"
    Device         "GPU0"
    Monitor        "Monitor0"
    DefaultDepth    24
    Option         "MetaModes" "HDMI-0: 1920x1080_60 +0+0"
    Option         "ConnectedMonitor" "HDMI-0"
    SubSection     "Display"
        Depth       24
        Modes      "1920x1080"
        Virtual    1920 1080
    EndSubSection
EndSection

Section "Monitor"
    Identifier     "Monitor0"
    VendorName     "Virtual"
    ModelName      "Virtual 1080p"
    HorizSync       30.0 - 81.0
    VertRefresh     56.0 - 76.0
    Option         "DPMS" "false"
    Option         "PreferredMode" "1920x1080"
EndSection
```

**File 2:** `/etc/X11/edid-1080p.bin` — 128-byte binary EDID declaring a 1920x1080@60Hz monitor. Generated programmatically with valid checksum.

**Both files persisted via `batocera-save-overlay`** so they survive reboots.

**Result:** `xrandr` shows `HDMI-0 connected primary 1920x1080+0+0` with a clean mode list (1920x1080, 1024x768, 800x600, 640x480 only). No more 8K phantom resolutions.

### Sunshine Web UI

Access at: `https://192.168.1.187:47990`
- Set username + password on first visit
- Add application: **EmulationStation** (command: `emulationstation`)

### Moonlight Client

Install Moonlight on any client device (Android, iOS, Windows, Fire TV, etc.).
- Add host: `192.168.1.187`
- Pin code shown in Sunshine web UI on first pair
- Resolution/bitrate: start at 1080p/30Mbps on LAN, tune up from there

### GTX 1650 + Sunshine

GTX 1650 supports **NVENC** hardware encoding — this is what makes streaming low-latency.
Encoder set to `nvenc` in sunshine.conf (also configurable via web UI).

---

## Step 9 — Saves & Backup

- Saves stored in `/userdata/saves/` — on the 30GB OS drive
- Periodic backup: `rsync -av /userdata/saves/ /media/ROMS-OTHER/saves-backup/`
- Add as a cron via `Main Menu → System Settings → Advanced → Cron`

---

## SSH Quick Reference

```bash
ssh root@[batocera-ip]          # connect
batocera-save-overlay           # persist any manual file changes to /etc or /usr
emulationstation --no-splash    # restart ES from CLI
systemctl restart emulationstation
```

---

## Key batocera.conf Settings Applied

```ini
global.videomode=1920x1080       # required — auto-detect was broken
system.hardware.gpu=nvidia        # required — must be set manually
system.dns=8.8.8.8               # temp until t620 AdGuard is online
scraper=thegamesdb               # personal API key configured
# Resolution upscaling
gamecube.dolphin_internal_resolution=2
wii.dolphin_internal_resolution=2
ps2.pcsx2_upscale_multiplier=2
psx.renderer=vulkan
psx.resolution=2
gba.mgba_gb_internal_framebuffer_scale=4
n64.gfxplugin_resolution=2
```

---

## Open Items

- [x] ~~Label Samsung USB drives~~ — labeled ROMS-PS2 → Games_II, ROMS-GC → Games_II, ROMS-OTHER → Games_I
- [x] ~~PS2 BIOS~~ — `ps2-0230a-20080220.bin` uploaded and verified
- [x] ~~PS1 BIOS~~ — `scph5501.bin` uploaded and verified
- [x] ~~DNS configured~~ — set to 8.8.8.8 (temp)
- [x] ~~Scraper configured~~ — TheGamesDB with personal API key
- [x] ~~Resolution upscaling~~ — set globally for all systems
- [ ] GameCube IPL.bin — `bios/GC/USA/IPL.bin` (suppresses BIOS warning)
- [ ] ScreenScraper account (Daddyboo) — website login broken, API 403. Try fresh registration.
- [ ] Switch `system.dns` back to t620 IP once AdGuard is online
- [ ] ROM acquisition: Retrode (~$80) vs GB Operator (~$50) for cartridge dumping
- [x] ~~Sunshine~~ — installed as Flatpak, config locked to 1080p/NVENC, headless display fix applied via xorg + custom EDID
