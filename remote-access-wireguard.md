# Remote Access — Pre-Departure Setup Plan

**Departure:** Friday
**Duration:** 10 days
**Goal:** Access Claude Code on WORKSTATION and Unraid server from Linux laptop over WireGuard VPN

---

## Architecture

```
Linux Laptop
  └── WireGuard VPN → HSTNC (VPN endpoint)
        ├── SSH → WORKSTATION (Claude Code for PC projects)
        └── SSH → Unraid Server (Claude Code Docker for server projects)
```

---

## Tuesday Night

**Do first — sequential (fast, must be done before parallelizing):**
- [ ] WORKSTATION: Install OpenSSH Server
  ```powershell
  Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
  Start-Service sshd
  Set-Service -Name sshd -StartupType Automatic
  New-NetFirewallRule -Name sshd -DisplayName 'OpenSSH Server' -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 22
  ```
- [ ] WORKSTATION: Note LAN IP (`ipconfig`), test SSH from another device on LAN
- [ ] Flash all three USB boot drives:
  - SanDisk Ultra Fit → Unraid boot drive
  - Cruzer Snap #1 → t620 (AdGuard)
  - Cruzer Snap #2 → HSTNC (WireGuard)
- [ ] Unraid: First boot, purchase license, assign drives, **start parity build**

**Parallel — while parity builds overnight:**

*HSTNC (WireGuard VPN):*
- [ ] Boot Linux installer, install OS
- [ ] `sudo apt install wireguard`
- [ ] Generate server keypair, write `wg0.conf`
- [ ] Open UFW port 51820 UDP
- [ ] `sudo systemctl enable --now wg-quick@wg0`
- [ ] Generate laptop peer config, export `.conf`
- [ ] Port forward UDP 51820 on router → HSTNC LAN IP

*t620 (AdGuard Home):*
- [ ] Boot Linux installer, install OS
- [ ] Install Docker
- [ ] Deploy AdGuard Home container
- [ ] Configure router DNS → t620 LAN IP
- [ ] Verify ad blocking from browser

*Linux Laptop:*
- [ ] Install distro, update packages
- [ ] `sudo apt install wireguard remmina code tmux`
- [ ] Drop HSTNC peer `.conf` into `/etc/wireguard/wg0.conf`
- [ ] VS Code: install Remote-SSH extension (`ms-vscode-remote.remote-ssh`)
- [ ] Add WORKSTATION to `~/.ssh/config`:
  ```
  Host workstation
      HostName <WORKSTATION-LAN-IP>
      User ryan
  ```
- [ ] Test VS Code remote connection to WORKSTATION
- [ ] Test tmux session attach/detach

*WORKSTATION:*
- [ ] Install tmux in WSL
- [ ] Enable Remote Desktop: Settings → System → Remote Desktop → Enable

---

## Wednesday

**Unraid (parity done or nearly done):**
- [ ] Create shares: `media/`, `photos/`, `appdata/`
- [ ] Install WireGuard via Community Apps, configure peer, export laptop `.conf`
  - Note: if using HSTNC as VPN endpoint this is not needed — HSTNC handles tunneling
- [ ] Install Jellyfin container, configure Arc A310 QSV
- [ ] Install Claude Code Docker container:
  ```bash
  docker run -it \
    --name claude-code \
    -v /mnt/user:/workspace \
    -e ANTHROPIC_API_KEY=your_key_here \
    ghcr.io/anthropics/claude-code
  ```
- [ ] Install Immich, point at `/mnt/user/photos/`

**Tunnel + end-to-end (while containers spin up):**
- [ ] `sudo wg-quick up wg0` on laptop
- [ ] SSH to WORKSTATION through VPN tunnel
- [ ] SSH to Unraid server through VPN tunnel
- [ ] **Test from phone LTE — not home WiFi (critical)**

---

## Thursday — Full End-to-End Verification

- [ ] VPN up → SSH to WORKSTATION → Claude Code in tmux → detach/reattach
- [ ] VPN up → SSH to Unraid → Claude Code Docker → detach/reattach
- [ ] RDP to WORKSTATION via Remmina (fallback test)
- [ ] AdGuard confirmed blocking on laptop through tunnel
- [ ] Fix anything broken before packing

---

## On the Road — Quick Reference

| Task | Command |
|------|---------|
| Connect VPN | `sudo wg-quick up wg0` |
| Disconnect VPN | `sudo wg-quick down wg0` |
| SSH to WORKSTATION | `ssh workstation` |
| SSH to Unraid | `ssh root@<SERVER-IP>` |
| New tmux session | `tmux new -s claude` |
| Attach tmux session | `tmux attach -t claude` |
| Detach tmux | `Ctrl+B then D` |
| Open WORKSTATION desktop | Remmina → RDP → `<WORKSTATION-IP>` |

---

## Key Notes
- **Parity build is the long pole** — start it as early as possible Tuesday, everything else runs in parallel
- **Test WireGuard from phone LTE**, not home WiFi — LTE is the only true external test
- **Port forward UDP 51820** on router → HSTNC LAN IP — most common miss
- **tmux on both machines** — protects sessions from connection drops while traveling
