# Homelab Build Guides

Working build documentation for an Unraid-based homelab stack, plus PC build
references that pair with it. Each guide is a real build that was completed (or
is in active use), with the parts list, the architectural choices, the BIOS
settings that mattered, and the gotchas that cost time the first time around.

## What's inside

### Server / homelab guides

| Guide | What it covers |
|---|---|
| **[unraid-server-build.md](unraid-server-build.md)** | Full AM4 / Ryzen 7 Unraid build — parts, PCIe layout, BIOS, headless setup, boot drive |
| **[unraid-storage-strategy.md](unraid-storage-strategy.md)** | Parity drive selection, cache pool layout, share-by-share cache settings, USB-rotation backup strategy |
| **[services-and-security.md](services-and-security.md)** | Jellyfin (with Intel Arc QSV transcoding), Immich, AdGuard (on separate node), AutoRipper pipeline, firewall + Tailscale architecture |
| **[batocera-retro-gaming.md](batocera-retro-gaming.md)** | Repurposed-PC retro gaming rig — Batocera install, ROM org, Moonlight + Sunshine streaming, Nvidia driver setup, headless EDID fix |
| **[alpine-backup-dns.md](alpine-backup-dns.md)** | Alpine Linux + dnsmasq as a backup DNS resolver on a fanless ThinkCentre/HP thin client. Forwards to primary AdGuard, falls back to Cloudflare. |
| **[remote-access-wireguard.md](remote-access-wireguard.md)** | WireGuard VPN + SSH setup for remote access while traveling. Pre-departure checklist that actually works. |

### Hardware reference (`hardware/`)

| Guide | What it covers |
|---|---|
| **[am4-platform-reference.md](hardware/am4-platform-reference.md)** | AM4 platform inventory — CPU, chipset, board, RAM, PCIe behavior. Reference for the Unraid build above. |
| **[am5-workstation-build.md](hardware/am5-workstation-build.md)** | Ryzen 9 7900X / DDR5 / RTX-class workstation build — parts, EXPO, NVMe layout |
| **[gpu-riser-cable-research.md](hardware/gpu-riser-cable-research.md)** | Detailed PCIe 4.0 riser cable research — what actually works at full bandwidth and what silently downgrades. Was painful to find first time. |
| **[pc-build-use-cases.md](hardware/pc-build-use-cases.md)** | The use cases driving the workstation build — daily driver, video editing, gaming, light AI inference — with concrete spec implications |
| **[streaming-pc-build.md](hardware/streaming-pc-build.md)** | A separate streaming/encoding PC build for offloading transcode/encode work |

## Why this exists

Most homelab build documentation online falls into two camps:
1. **Marketing-flavored "my build" posts** that skip the hard parts (riser cable
   compatibility, IOMMU behavior, BIOS settings that broke things, drive labels
   that don't match what the OS expects).
2. **Forum threads** that have all the real information buried in 40 pages of replies.

This repo is the format I wished existed when I was building: short, opinionated,
with rationale for non-obvious choices and links to where decisions were tested.

## Conventions used throughout

- **Placeholders use `<like-this>`** — wherever you see `<NAS-IP>` or `<YOUR-TAILNET>` you fill in your own.
- **IP examples use `192.168.1.x`** — the example LAN range. Tailscale CGNAT IPs use `100.x.x.NN`.
- **No real credentials anywhere.** Where a doc shows where you put a password, it's labeled `YOUR_PASSWORD`.
- **Status as of writing** at the top of each doc. Things change — verify before you copy.

## Related repos

- [autoripper-toolkit](https://github.com/Booboogotrizz/autoripper-toolkit) — the disc-rip pipeline the services guide references
- [homelab-cheatsheet-overlay](https://github.com/Booboogotrizz/homelab-cheatsheet-overlay) — the desktop overlay that shows the IPs/services from these guides
- [ventoy-multiboot-toolkit](https://github.com/Booboogotrizz/ventoy-multiboot-toolkit) — the USB toolkit referenced in the recovery sections
- [claude-project-dashboard](https://github.com/Booboogotrizz/claude-project-dashboard) — what I use to track which builds are at what completion state

## License

CC BY-SA 4.0 — copy, adapt, build your own homelab. Attribution appreciated.
