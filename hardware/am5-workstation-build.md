# AM5 Build — Specs & Configuration

**Status:** Active / In Progress
**Last Updated:** March 2026
**Platform:** AMD AM5

---

## Core Components

| Component | Details |
|-----------|---------|
| **CPU** | AMD Ryzen 9 7900X — 12-core / 24-thread, 4.7GHz base |
| **Motherboard** | ASUS TUF GAMING X870-PLUS WIFI (Rev 1.xx) |
| **RAM** | 64GB DDR5 — Corsair Vengeance CMH64GX5M2Y6400C32 (2x32GB) |
| **GPU** | Gigabyte Aorus OC 2080 Ti 11GB (Water Block) |
| **PSU** | EVGA SuperNOVA P3 1000W (80+ Platinum) |
| **Case** | Thermaltake P90 — Open Air Tower |
| **Cooling** | Custom Soft Tube Loop — Thermaltake Pacific line |

---

## Storage

| Letter | Drive | Type | Capacity | Bus | Firmware | Role |
|--------|-------|------|----------|-----|----------|------|
| C:\ | Crucial T710 (CT1000T710SSD8) | NVMe SSD | 1TB | PCIe Gen 5 | PBCR5103 | OS + Apps |
| D:\ | Sabrent Rocket 4.0 Plus | NVMe SSD | 1TB | PCIe Gen 4 | RKT4P1.2 | Auxiliary |
| E:\ | Samsung 990 EVO Plus | NVMe SSD | 1TB | NVMe Gen 4/5 | 2B2QKXG7 | Auxiliary |
| F:\ | KingSpec Z5-2TB | External SSD | 2TB | USB-C | 1000 | PC Game Library (labeled "Games") |
| H:\ | SanDisk Cruzer | USB Flash | 16GB | USB-A | 1.00 | Utility |
| G:\ | Optical Drive Bay | — | — | SATA | — | MakeMKV / Ripping |

**Notes:**
- Crucial T710 Gen 5 = fastest, correctly used as OS boot
- KingSpec Z5 = dedicated external USB-C game library
- Sabrent + Samsung = available for installs, scratch, media staging

---

## Memory

- **Kit:** Corsair Vengeance DDR5 — CMH64GX5M2Y6400C32
- **Capacity:** 64GB (2x32GB)
- **Rated / Configured Speed:** DDR5-6400 — XMP/EXPO confirmed active at 6400MHz
- **Slots:** P0 Channel A + P0 Channel B — dual channel confirmed
- **Timings:** Check HWiNFO64 → SPD tab for primary timings (CL/tRCD/tRP/tRAS)

---

## GPU

- **Model:** NVIDIA GeForce RTX 2080 Ti (Gigabyte Aorus OC WB)
- **VRAM:** 11GB GDDR6
- **Driver Version:** 32.0.15.9174 (Dec 30, 2025)
- **Cooling:** Gigabyte water block — in custom loop with CPU
- **PCIe Connection:** Riser cable (installed)
- **OC Profile:** Check MSI Afterburner for active profile

---

## Cooling — Custom Soft Tube Loop (Thermaltake Pacific)

- **Origin Kit:** Thermaltake Pacific 360 Soft Tube Kit (parts replaced/expanded over time)
- **Radiator:** 360mm
- **Fan Config:** Push-Pull — 3x front + 3x rear of rad (6 fans total)
- **Pump/Res 1:** Thermaltake Pacific — Large combo unit
- **Pump/Res 2:** Thermaltake Pacific — Small combo unit
- **Fittings:** Thermaltake Pacific line
- **Tubing:** Soft tube (Thermaltake Pacific spec)
- **Coolant:** XSPC EC6 Clear UV — approx. 2.5L total loop capacity
- **Components in Loop:** CPU + GPU (2-block loop)
- **CPU Temps Under Load:** [Check HWiNFO64]
- **GPU Temps Under Load:** [Check HWiNFO64]

---

## Case — Thermaltake P90

- **Type:** Open Air Tower
- **Rad Mount:** Front-mounted 360mm
- **Active Fans:** 6x push-pull on 360 rad
- **Note:** Open air — excellent airflow, watch for dust in home workshop environment

---

## Displays

| # | Model | Resolution | Size | Panel | Refresh | Position |
|---|-------|------------|------|-------|---------|----------|
| 1 — Primary | MSI MAG321CQR | 2560x1440 | 32" | VA Curved | 144Hz | Center |
| 2 — Aux | Sceptre F22 (SPT083E) | 1920x1080 | 22" | — | — | Above primary |
| 3 — Aux | HP VH240a (HPN3499) | 1920x1080 | 24" | IPS | 75Hz | Right |
| 4 — Spare | [Unused — offline] | 1080p | — | — | — | — |

---

## Peripherals

| Device | Model | Notes |
|--------|-------|-------|
| **Keyboard (primary)** | Redragon K686 Pro RGB | Wired, Hall Effect magnetic switches |
| **Keyboard (spare)** | Razer Huntsman Mini | Offline — previous keyboard |
| **Mouse** | Razer Basilisk V2 | Wired |
| **Mousepad** | Razer Goliathus Chroma | RGB mousepad |
| **Drawing Tablet** | Wacom Intuos CTH-480 | Small, USB |
| **Stream Deck** | Elgato Stream Deck XL | USB |
| **Headset (gaming)** | HP Omen Headset | Wired, primary for gaming |
| **Speakers** | Soundbar | Primary for general audio / media |
| **Mic** | Elgato Wave:3 | USB condenser, managed via Wave Link |

---

## Audio Stack

- **Mic:** Elgato Wave:3 (USB condenser)
- **Software:** Elgato Wave Link 2.0.6 + Voicemod installed
- **Gaming audio:** HP Omen headset
- **General audio:** Soundbar
- **Onboard audio:** Realtek HD Audio (available as fallback)
- **GPU audio:** NVIDIA HD Audio (for HDMI/DP passthrough)

---

## Motherboard & BIOS

- **Model:** ASUS TUF GAMING X870-PLUS WIFI
- **Chipset:** AMD X870
- **BIOS Version:** 0816 (AMI)
- **Form Factor:** ATX
- **WiFi:** MediaTek Wi-Fi 7 MT7925 (802.11be)
- **LAN:** Realtek PCIe 2.5GbE

### BIOS Settings — Confirmed via Windows

| Setting | Status |
|---------|--------|
| XMP/EXPO DDR5-6400 | ✅ Active |
| AMD-V / SVM Virtualization | ✅ Enabled |
| Secure Boot | ⚠️ Unknown — likely disabled |
| Resizable BAR | ⚠️ Unknown — fill in manually |

### BIOS Settings — Fill In Manually (Next BIOS Visit)

| Setting | Your Value |
|---------|-----------|
| PBO Mode | **Advanced** (confirmed 2026-04-20) |
| PBO Scalar | [FILL IN — check next BIOS visit] |
| PBO Limits | [Motherboard / Manual / Unlimited — check next BIOS visit] |
| Curve Optimizer | **Enabled** (confirmed 2026-04-20) — per-core or all-core offset values need to be captured from BIOS |
| CPU Core Voltage | [Auto / Offset / Override — check next BIOS visit] |
| FCLK Frequency | [Auto or manual MHz] |
| UCLK = MCLK | [Coupled / Decoupled] |
| DRAM Primary Timings | [CL-tRCD-tRP-tRAS] |
| Memory Context Restore | [Enabled / Disabled — target to ENABLE for faster boot] |
| Resizable BAR | [Enabled / Disabled] |
| Above 4G Decoding | [Enabled / Disabled] |
| Secure Boot | [Enabled / Disabled] |
| Fast Boot | [Enabled / Disabled] |
| CSM | [Disabled recommended on AM5] |

**OC Status Summary (2026-04-20):**
- **CPU:** PBO Advanced + Curve Optimizer enabled. Exact CO values TBD (capture next BIOS visit). System stable, no instability reported.
- **RAM:** EXPO DDR5-6400 only. No manual timing/voltage tuning. → Safe for MCR enable.
- **GPU:** MSI Afterburner OC profile exists (2080 Ti). Unrelated to BIOS/MCR.

**Tip:** Save a BIOS profile to USB — BIOS → Tool → Profiles → Save to USB. Creates a full binary backup you can restore from.

---

## Key Driver Versions

| Component | Version | Date |
|-----------|---------|------|
| NVIDIA GPU | 32.0.15.9174 | Dec 30, 2025 |
| AMD Chipset | 7.11.26.2142 | — |
| AMD Ryzen Balanced | 8.0.0.13 | — |
| Realtek 2.5GbE | 1125.27.50.919 | Sep 2025 |
| MediaTek Wi-Fi 7 | 5.7.0.4669 | Sep 2025 |
| AMD SMBus | 5.12.0.44 | Sep 2025 |

---

## Networking

- **Primary NIC:** SFP+ card — 10GbE DAC cable to AM4 server and NICGIGA switches
- **Onboard NIC:** Realtek PCIe 2.5GbE — not in use (SFP+ is primary)
- **Wireless:** MediaTek Wi-Fi 7 MT7925 — Wi-Fi 7 / 6GHz capable

---

## Software Stack

| Category | Software | Version |
|----------|---------|---------|
| OS | Windows 11 Pro | Build 26200 |
| 3D Modeling | Blender | 5.0.1 |
| Video Encoding | HandBrake | 1.10.2 |
| Disc Ripping | MakeMKV | 1.18.3 |
| GPU OC / Monitor | MSI Afterburner | 4.6.6 |
| System Monitor | HWiNFO64 | 8.42 |
| GPU Info | GPU-Z | 2.69.0 |
| Stream Control | Elgato Stream Deck | 7.2.1 |
| Audio Mix | Elgato Wave Link | 2.0.6 |
| Voice Processing | Voicemod | Installed |
| AMD Platform | AMD Settings | 2026.0108 |
| Keyboard Software | Redragon K686 | V2.06.01 |

---

## Use Cases

- **Gaming:** 1440p / 144fps on MSI MAG321CQR
- **3D Modeling:** Blender 5.0.1
- **Video Encoding:** HandBrake (transcoding ripped media)
- **Disc Ripping:** MakeMKV via optical drive (G:\)
- **Streaming / Recording:** Stream Deck XL + Wave:3 mic
- **Tablet / Drawing:** Wacom Intuos CTH-480

---

## Open Items

- [ ] GPU riser cable — already installed, identify make/model for spec doc
- [ ] Document BIOS settings (PBO, ReBAR, DRAM timings) — next BIOS visit
- [ ] Log CPU + GPU loop temps under full load via HWiNFO64
- [ ] Identify spare 4th 1080p monitor model
- [ ] Note Sceptre F22 and HP VH240a refresh rates
