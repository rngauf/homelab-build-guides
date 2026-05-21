# Unraid Server — Hardware & OS Reference Build

A reference Unraid server build on AM4 / Ryzen 7 — the parts list, PCIe layout,
storage roles, and BIOS settings that worked. Use as a starting point and adjust
to your case/budget/use case.

**Platform:** AM4 (Ryzen 7 3700X)

---

## Hardware Summary

| Component | Details |
|-----------|---------|
| **CPU** | AMD Ryzen 7 3700X — 8c/16t, 3.6GHz base |
| **Motherboard** | ASRock B550M Pro4 — micro-ATX, AM4, DDR4 |
| **RAM** | 16GB DDR4 |
| **GPU** | Intel Arc Sparkle A310 — transcoding only |
| **NIC** | SuperMicro AOC-STGN-I2S — dual-port 10GbE SFP+ |
| **CPU Cooler** | AMD Wraith Prism |
| **PSU** | 750W |
| **Case** | Custom 3D printed mini server |

---

## Operating System

- **OS:** Unraid
- **License:** Purchasing on build completion (~$60)
- **Boot Drive:** SanDisk Ultra Fit USB 3.1 (28GB) — USB-only boot as required by Unraid
- **Headless:** Yes — managed via web UI
- **Web UI Default Port:** 80 (http) / [assign static IP on first boot]
- **Default Hostname:** tower (Unraid default — rename as desired)

---

## PCIe & M.2 Layout

| Slot | Installed | Bandwidth |
|------|-----------|-----------|
| PCIE1 x16 (CPU) | Intel Arc Sparkle A310 | Full PCIe 4.0 x16 |
| PCIE2 x1 (chipset) | Empty | PCIe 3.0 x1 |
| PCIE3 x16 physical / x4 electrical | SuperMicro SFP+ NIC | PCIe 3.0 x4 — fine for 10GbE |
| M2_1 (CPU, PCIe Gen4 x4) | 256GB NVMe — Docker appdata + Claude/AI server files | Fast NVMe, not in array |
| M2_2 (chipset, PCIe Gen3 x2) | M.2 to SATA expansion | +6 SATA ports; installs in M2_2, causes onboard SATA 5 & 6 to go offline (PCIe lane conflict). Workaround: use 4 of 6 adapter ports. Active SATA: 4 onboard + 4 adapter = 8 total. |

**Total SATA ports:** 8 active (4 onboard ports 1–4 + 4 of 6 adapter ports; onboard ports 5 & 6 disabled due to M.2 adapter PCIe lane conflict)

---

## Network

- **Primary Link:** 10GbE SFP+ DAC cable to main AM5 PC
- **Switch:** 2x NICGIGA hybrid switches (2x 10GbE SFP+ + 4x 2.5GbE RJ45 per switch)
- **Assigned IP:** [FILL IN on first boot — set static via router DHCP reservation]
- **Hostname:** [FILL IN — set in Unraid first boot]
- **Link Speed to Main PC:** 10Gbps
- **Onboard NIC:** 1GbE — management / fallback

---

## Remote Access

- **Primary:** Unraid web UI (LAN)
- **Remote Access Plan:** [FILL IN — Tailscale VPN recommended for secure remote]
- **SSH:** Available via Unraid terminal

---

## Additional Nodes on Network

| Device | Hardware | Role |
|--------|---------|------|
| t620 Plus (replacement) | HP t620 Plus (quad-core AMD GX) | AdGuard Home + WireGuard VPN + Uptime Kuma — ordered, in transit (original t620 dead, won't power on) |
| dns2 (HSTNC) | HP HSTNC-006-TC (Intel Atom N280) | Backup DNS — Alpine Linux x86, dnsmasq → t620 AdGuard, static 192.168.1.5 |
| Retro Gaming Rig | i7-4770 / Asus B85M-E / GTX 1650 / 32GB DDR3 | Batocera — physically integrated into server stack |
| STREAM-PC | AM4 build — specs TBD | Old streaming PC — future role TBD. See `Reference\Hardware-PC-Builds\stream-pc-specs.md` |
| **thin-client Terminal** | HP thin-client Mobile Thin Client | Remote terminal for WORKSTATION — Ubuntu install in progress (see RESUME HERE below) |

**Note:** AdGuard, WireGuard, and Uptime Kuma consolidated on t620 (dedicated always-on, separate from WORKSTATION).

---

## HP thin-client — Remote Terminal Build

**Hardware:**
- CPU: AMD PRO A8-8600B (4C)
- RAM: 8GB DDR3 mismatched (single-channel, working)
- Storage: 32GB NVMe (`/dev/sda`) + 500GB HDD
- BIOS: Legacy (MBR)

**Planned role:** Remote terminal — VS Code Remote SSH into WORKSTATION for Claude Code work

**OS:** Ubuntu 24.04 LTS via Ventoy USB

---

## RESUME HERE — 2026-03-26

**Status:** Ubuntu install in progress — bootloader not installing correctly to NVMe

**Problem:** Ubuntu kept booting from Ventoy USB instead of NVMe. GRUB not writing to `/dev/sda` (NVMe).

**Last attempted fix:** Manual partitioning with:
- `/dev/sda` (32GB NVMe) — 4GB swap + ~27GB ext4 `/` with bootable flag
- "Device for boot loader installation" set to `/dev/sda`

**Next steps:**
1. Complete the install attempt with manual partitioning as described above
2. On reboot without USB — if it still fails, boot Ventoy USB → Try Ubuntu → open terminal → run `boot-repair`
3. After successful boot: install VS Code, set up Remote SSH to WORKSTATION
4. Mount 500GB HDD at `/home` or `/data`

---

## Build Checklist

- [x] Finish 3D printing case
- [x] Assemble all components
- [ ] Install Unraid on SanDisk Ultra Fit USB boot drive
- [ ] Purchase Unraid license
- [ ] Assign static IP / hostname
- [ ] Configure array and cache pool (see storage doc)
- [ ] Migrate Samsung T7 data to protected array (priority #1 on day one)
- [ ] Install Docker services (Jellyfin, Immich)
- [ ] Configure AutoRipper pipeline
- [ ] Set up AdGuard + WireGuard + Uptime Kuma on HP t620
