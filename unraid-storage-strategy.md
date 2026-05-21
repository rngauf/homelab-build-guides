# Homelab — Storage & Snapshot Strategy (Unraid)

**OS:** Unraid

---

## Drive Inventory & Roles

| Drive | Type | Size | Unraid Role |
|-------|------|------|-------------|
| Toshiba HDD #1 | HDD | 3TB | Parity (protects entire array) |
| Toshiba HDD #2 | HDD | 3TB | Array — media storage |
| Toshiba HDD #3 | HDD | 3TB | Array — media storage |
| Additional HDD | HDD | 500GB | Array — media / general |
| WD SSD #1 | SSD | 500GB | Cache pool |
| WD SSD #2 | SSD | 500GB | Cache pool |
| WD SSD #3 | SSD | 500GB | Cache pool |
| SanDisk Ultra Fit | USB Flash | 28GB | Unraid OS boot drive (USB — required by Unraid) |
| 256GB NVMe | NVMe SSD | 256GB | Docker appdata + Claude/AI server files (M2_1, CPU-direct, not in array) |
| Samsung T7 | USB-C SSD | 1TB | Migrate to array day 1 — family photos/docs |
| USB Drive #1 | USB Thumb Drive (USB 2.0) | 2TB | Backup target |
| USB Drive #2 | USB Thumb Drive (USB 2.0) | 2TB | Backup target |
| USB Drive #3 | USB Thumb Drive (USB 2.0) | 2TB | Backup target |

---

## Unraid Array Configuration

### Parity
- **Drive:** 1x 3TB Toshiba HDD
- **Protection:** Single parity — can survive 1 drive failure
- **Rebuild time:** ~8-12 hours for 3TB

### Data Array
- **Drives:** 2x 3TB HDD + 1x 500GB HDD = ~6.5TB usable
- **Total usable array:** ~6.5TB

### Cache Pool
- **Drives:** 3x 500GB WD SSD = 1.5TB
- **Mode:** [FILL IN — RAID0 for max space / RAID1 mirror two drives for protection]
- **Purpose:** Fast writes for VM images, Docker app data, active downloads before moving to array
- **Note:** If using RAID0 on cache, data written here but not yet flushed to array could be lost if SSDs fail — consider mirroring 2 of the 3 for safety

### Backup Tier
- **3x 2TB USB thumb drives (USB 2.0) = 6TB backup capacity**
- Used as Unraid backup targets via Appdata Backup plugin or built-in tools
- Not part of the protected array — USB not recommended in Unraid array

---

## Total Storage Summary

| Tier | Capacity | Contents |
|------|----------|---------|
| Protected array | ~6.5TB | Media library, family files, documents |
| Cache pool | 1.5TB | Active app data, VMs, fast scratch |
| Backup | 6TB | Automated backups of array |
| **Total usable** | **~9TB** | Across all tiers |

---

## Projected Storage Usage

| Data Type | Current | Long-term |
|-----------|---------|-----------|
| Family photos/docs (from T7) | ~650GB | ~2-3TB |
| DVD collection (400 discs @ ~2.5GB H.265 1080p) | ~1TB | ~12.5TB at 5,000 discs |
| Blu-ray collection (100 discs @ ~6GB H.265) | ~600GB | Growing |
| Docker/VM/app data (cache) | ~1.5TB | ~1.5TB |
| **Total current** | **~3.75TB** | Expanding |

**Note:** Current collection fits comfortably. Plan to add larger drives (8TB+) when array hits 70% full.

---

## Share / Folder Structure (Unraid Shares)

```
/mnt/user/
├── media/
│   ├── movies/          ← Jellyfin library
│   │   └── Movie Title (Year)/Movie Title (Year).mkv
│   └── tv/              ← Jellyfin library
│       └── Show Name/Season XX/
├── photos/              ← Immich library (family photos)
├── documents/           ← Document sync
├── appdata/             ← Docker container config/data (cache preferred)
├── system/              ← Unraid VM and system files (cache)
└── backups/             ← Backup landing zone
```

---

## Snapshot / Backup Strategy (Unraid)

Unraid doesn't use ZFS snapshots natively. Backup strategy uses plugins + scheduled tasks:

### Appdata Backup (Docker configs)
- **Plugin:** CA Appdata Backup
- **Frequency:** Daily
- **Destination:** /backups/appdata/ on array
- **Retention:** 7 days

### Media Backup
- Media (Jellyfin library) is ripped from physical discs — re-rippable
- **Priority:** Low — can be re-created from discs
- **Backup:** Optional, space permitting, to USB drives

### Family Photos / Documents — HIGHEST PRIORITY
- **Day 1 task:** Copy all T7 content to /photos/ and /documents/ shares
- **Backup:** Automated nightly copy to USB backup drives
- **3-2-1 Goal:**
  - Copy 1: Protected Unraid array (parity)
  - Copy 2: USB backup drives (on-site)
  - Copy 3: Samsung T7 (keep as offsite/portable backup)

### USB Backup Schedule
- **Tool:** Unraid built-in backup or CA Backup plugin
- **Frequency:** Nightly — photos and documents
- **Retention:** Keep last 3 copies

---

## Expansion Plan

When array hits 70% full:
- Add one 8TB+ HDD to the array (Unraid — just plug in and add, no rebuild required)
- 8TB drives ~$100-150 — cost effective expansion
- Upgrade parity drive to match if new data drive is larger than current parity

---

## Open Items

- [ ] Decide cache pool mode — RAID0 (1.5TB) or mirror 2 SSDs (500GB protected + 500GB extra)
- [ ] Choose Unraid backup plugin (CA Backup vs built-in)
- [ ] Assign static IP before first use
- [ ] Migrate T7 data on day 1 — before anything else
- [ ] Set up nightly backup schedule for photos/docs
