# AM4 Platform — Server Hardware Inventory

**Role:** Home server / NAS / media server
**Platform:** AMD AM4

---

## Core Components

| Component | Details |
|-----------|---------|
| **CPU** | AMD Ryzen 7 3700X — 8-core / 16-thread, 3.6GHz base |
| **Motherboard** | ASRock B550M Pro4 (micro-ATX, AM4, DDR4, PCIe 4.0) |
| **RAM** | 16GB DDR4 |
| **CPU Cooler** | AMD Wraith Prism (stock) |
| **GPU** | Intel Arc Sparkle A310 — dedicated transcoding card |
| **NIC** | SuperMicro AOC-STGN-I2S — dual-port 10GbE SFP+ (Intel 82599) |
| **PSU** | 750W |
| **Case** | Custom 3D printed mini server — complete |

---

## PCIe Slot Layout

| Slot | Card | Notes |
|------|------|-------|
| PCIE1 — x16 (PCIe 4.0, CPU-direct) | Intel Arc Sparkle A310 | Jellyfin transcoding |
| PCIE2 — x1 (PCIe 3.0) | Empty | Available |
| PCIE3 — x4 (PCIe 3.0) | SuperMicro SFP+ NIC | 10GbE link to main PC |
| M2_1 — PCIe Gen4 x4 (CPU) | 256GB NVMe SSD | Docker appdata + Claude/AI server files |
| M2_2 — PCIe Gen3 x2 (chipset) | M.2 to SATA expansion card | +6 SATA ports (see note below) |

### M.2 SATA Adapter — Known Conflict

The M.2-to-SATA adapter in M2_2 (chipset, PCIe Gen3 x2) causes onboard SATA ports 5 & 6 to go offline (PCIe lane conflict on B550M Pro4). Workaround: use 4 of the 6 adapter ports instead of the dead onboard ports 5 & 6. All drives visible in Unraid with this configuration.

**Active SATA ports:** 4 onboard (ports 1–4) + 4 adapter ports = **8 active SATA connections total**

---

## Storage Drives

| Drive | Type | Capacity | Interface | Role |
|-------|------|----------|-----------|------|
| SanDisk Ultra Fit | USB Flash | 28GB | USB 3.1 | Unraid OS boot drive |
| 256GB NVMe | NVMe SSD | 256GB | M.2 PCIe Gen4 (M2_1) | Docker appdata + Claude/AI server files (fast NVMe, not in array) |
| Toshiba HDD #1 | HDD | 3TB | SATA | Parity drive |
| Toshiba HDD #2 | HDD | 3TB | SATA | Array — media |
| Toshiba HDD #3 | HDD | 3TB | SATA | Array — media |
| WD SSD #1 | SSD | 500GB | SATA | Cache pool |
| WD SSD #2 | SSD | 500GB | SATA | Cache pool |
| WD SSD #3 | SSD | 500GB | SATA | Cache pool |
| Additional HDD | HDD | 500GB | SATA | Array — media |
| Samsung T7 | External SSD | 1TB | USB-C | Family photos/docs — migrate to array Day 1 |
| USB Backup #1 | USB Thumb Drive | 2TB | USB 2.0 | Backup target |
| USB Backup #2 | USB Thumb Drive | 2TB | USB 2.0 | Backup target |
| USB Backup #3 | USB Thumb Drive | 2TB | USB 2.0 | Backup target |


---

## Networking

- **Primary NIC:** SuperMicro AOC-STGN-I2S — 10GbE SFP+ (Intel 82599)
- **Connection:** 10GbE SFP+ DAC cable → main PC AM5 (which also has matching SFP+ NIC)
- **Switch:** 2x NICGIGA hybrid switches (2x 10GbE SFP+ + 4x 2.5GbE RJ45 each)
- **Onboard NIC:** 1GbE (management / fallback)
- **IP Address:** [FILL IN — assign static on install]
- **Hostname:** [FILL IN — e.g., "tower" default in Unraid]

---

## GPU — Intel Arc Sparkle A310

- **Purpose:** Hardware transcoding for Jellyfin (not gaming)
- **VRAM:** 4GB
- **Codec support:** AV1, HEVC 10-bit, VP9 via Intel Xe Media Engine
- **Why Arc over GTX 1650:** Significantly better modern codec support for Jellyfin
- **Note:** GTX 1650 moved to Batocera retro gaming rig (i7-4770 build)

---

## Build Status

- [x] 3D printed case — complete
- [x] All components sourced and assembled
- [ ] Unraid license — purchase before trial expires (~$60)
- [ ] Reflash Ultra Fit with Unraid USB Creator (correctly — unplug all other USBs first)
- [ ] Investigate 2x WD SSDs not visible (M.2 adapter, resolved via 4-port workaround)

---

## Cross-Reference

For server configuration, services, and storage layout see: **Homelab & Server** project docs
