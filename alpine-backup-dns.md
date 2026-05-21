# DNS2 — Alpine Linux Backup DNS Setup

**Device:** HP HSTNC-006-TC
**Hostname:** dns2
**Role:** Backup DNS resolver — dnsmasq forwarding to t620 AdGuard Home, fallback to 8.8.8.8
**OS:** Alpine Linux Extended 3.23.3 x86 (32-bit — required, Atom N280 is 32-bit only)

---

## Network

| Field | Value |
|-------|-------|
| Static IP | 192.168.1.5 |
| MAC Address | XX:XX:XX:XX:XX:XX (reserved in router) |
| Gateway | 192.168.1.1 |
| Netmask | 255.255.255.0 |
| DNS (during setup) | 192.168.1.X (t620 AdGuard IP — fill in when t620 is configured) |

---

## Credentials

| Service | Username | Password |
|---------|----------|----------|
| Alpine root (SSH/console) | `root` | set during setup-alpine |

---

## Known Network Devices (from router)

| Hostname | IP | Notes |
|----------|----|-------|
| Router | 192.168.1.1 | Motorola C3510XZ |
| WORKSTATION | 192.168.1.50 | Main PC (Windows 11) — OpenSSH enabled |
| WORKSTATION | 192.168.1.51 | Second entry — possibly second NIC or stale lease |
| Tower | 192.168.1.10 | Unraid server |
| dns2 (HSTNC) | 192.168.1.5 | This device — DHCP reservation set |
| decoMeshXE75 | 192.168.1.102 | TP-Link Deco mesh node |
| Borg | 192.168.1.219 | Unknown device |
| t620 | TBD | AdGuard Home + WireGuard + Uptime Kuma |

---

## Hardware Specs

| Spec | Value |
|------|-------|
| CPU | Intel Atom N280, 1.66 GHz, single-core, **32-bit only** |
| RAM | 1–2 GB DDR3 (max 4 GB) |
| Storage | 2 GB internal flash + resident USB (Alpine install target) |
| Network | Gigabit Ethernet (wired only, no WiFi) |
| BIOS | Legacy AMI — no UEFI |
| Power | ~11–12W idle |

---

## Install Process

1. Boot from Alpine Extended x86 ISO on temp USB
2. Login: `root` (no password at live boot)
3. Run `setup-alpine`
4. Partition scheme: MBR (legacy BIOS only — Rufus: MBR + "Add fixes for old BIOSes")
5. Install target: resident USB (`sys` mode — full disk install)
6. Pull temp USB after install, reboot

**setup-alpine answers:**
- Keyboard: `us` / `us`
- Hostname: `dns2`
- Interface: `eth0`, static IP `192.168.1.5`
- Netmask: `255.255.255.0`
- Gateway: `192.168.1.1`
- DNS: t620 IP (AdGuard)
- Timezone: `US/Mountain`
- Proxy: none
- NTP: chrony (default)
- Mirror: fastest (f) or default
- SSH: openssh
- Disk: resident USB → `sys`

---

## dnsmasq Configuration

**Install:**
```bash
apk update
apk add dnsmasq
```

**`/etc/dnsmasq.conf`:**
```
no-resolv
server=192.168.1.X        # t620 AdGuard IP — fill in
server=8.8.8.8            # fallback if t620 is down
listen-address=127.0.0.1,192.168.1.5
cache-size=150
```

**Enable and start:**
```bash
rc-update add dnsmasq default
rc-service dnsmasq start
```

**Router:** Set secondary DNS to `192.168.1.5`

---

## DNS Failover Chain

```
All devices → t620 (192.168.1.X) AdGuard Home [primary, ad-blocking]
           → dns2 (192.168.1.5) dnsmasq → t620 [secondary, still ad-blocking]
           → 8.8.8.8 [last resort, no ad-blocking]
```