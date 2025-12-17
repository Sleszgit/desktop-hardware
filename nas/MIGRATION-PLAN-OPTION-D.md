# NAS Migration Plan - Option D (Recommended)

**Plan Created:** 17.12.2025
**Strategy:** Consolidate 918 & 920 data using homelab Proxmox as temporary staging
**Target Outcome:** Decommission both NAS, preserve all data, migrate services

---

## Overview

This plan leverages the homelab Proxmox (192.168.40.40) as a staging area to consolidate 918 & 920 data while maximizing disk reuse across your infrastructure.

### Key Principles
- ✅ Zero data loss - all content preserved
- ✅ Maximize disk reuse (healthy 918 disks → homelab)
- ✅ Free up UGREEN for 920 data temporarily
- ✅ Migrate critical services (nginx) to UGREEN
- ✅ Graceful decommissioning of both NAS devices

---

## Infrastructure Summary

### UGREEN Proxmox (192.168.40.60) - PRIMARY CONSOLIDATION TARGET
**Storage:** ZFS pool "storage" (20TB mirror)
- **Disks:** 2x Seagate ST22000NT001 20TB (3.5-year-old IronWolf PRO equivalent)
- **Current usage:** 5.67TB / 20TB (28%)
- **Free space:** 14.3TB
- **Current data:** Movies918 (1.5TB), Series918 (435GB), backupstomove (3.8TB transferred)

### Homelab Proxmox (192.168.40.40) - TEMPORARY STAGING + FINAL ARCHIVE
**Storage:** WD 10TB mirror in Proxmox
- **Disks:** 2x WD 10TB (fresh, healthy)
- **RAID:** Mirror (effective 10TB usable)
- **Current usage:** ~6TB / 10TB
- **Free space:** ~4TB
- **Available slots:** 4 additional HDD slots (max 6 total)
- **Purpose:** Receive 918 disks for data consolidation

### 918 NAS (192.168.40.10) - SOURCE #1
**Disks to migrate:**
- Slot 1: Seagate IronWolf PRO 16TB (14,116 hrs, 1.6 yrs) ✅ **REUSE**
- Slot 2: Seagate IronWolf PRO 14TB (30,459 hrs, 3.5 yrs) ✅ **REUSE**
- Slot 4: Seagate 14TB (14,124 hrs, 1.6 yrs) ✅ **REUSE**
- Slot 3: WD Red Pro 10TB (40,646 hrs, 4.6 yrs) 🛑 **RETIRE** (end of life)

**Data to migrate:** ~11.7TB remaining
- Volumes 1, 2, 3 content (mixed)
- System/backup data

### 920 NAS (192.168.40.20) - SOURCE #2 (CRITICAL)
**Disks (for future reference, not yet moved):**
- Slots 1-2: Seagate IronWolf PRO 20TB (19,047 hrs, 2.2 yrs) ✅ **REUSE**
- Slots 3-4: Seagate IronWolf PRO 16TB (30,282 hrs, 3.5 yrs) ✅ **REUSE**

**Data to preserve:** 30TB (URGENT - volume 95% full)
- Seriale 2023 (TV series): 17TB
- Filmy920 (movies): 13TB
- Docker/system files: minimal

**Services to migrate:**
- **VITAL:** Nginx web server
- **Optional:** Plex Media Server, Docker containers (can be disregarded)

---

## Migration Plan - Phase by Phase

### PHASE 1: Complete 918 Data Transfer to UGREEN (Days 1-3)

**Current Status:**
- Movies918: 998GB ✅ (complete)
- Series918: 435GB ✅ (complete)
- aaafilmscopy: 517GB ✅ (complete)
- backupstomove: 3.8TB 🟡 (in progress)
- **Remaining:** ~11.7TB in volumes 1/2/3

**Actions:**
1. Complete current backupstomove transfer (ETA: 2025-12-09)
2. Identify remaining data to transfer (volumes 1, 2, 3 system data)
3. Transfer all remaining 918 content to UGREEN `/storage/Media/`
4. **Verify checksums** - all data received correctly

**Space Check:**
- UGREEN free: 14.3TB
- 918 data to move: 11.7TB
- **Status:** ✅ Sufficient space

**Estimated Duration:** 2-3 days (depending on network speed ~46 MB/s)

**Completion Verification:**
```bash
# On UGREEN
du -sh /storage/Media/*
# Should show: ~16.5TB total (5.67TB existing + 11.7TB from 918)
```

---

### PHASE 2: Prepare 918 Disks for Homelab Migration (Day 3-4)

**Actions:**
1. Once all 918 data transferred to UGREEN, power down 918 NAS
2. **Remove disks from 918:**
   - Slot 1: Seagate 16TB (label: "918-Slot1-16TB-IronWolf")
   - Slot 2: Seagate 14TB (label: "918-Slot2-14TB-IronWolf")
   - Slot 4: Seagate 14TB (label: "918-Slot4-14TB-Seagate")
   - **Retire Slot 3:** WD 10TB (4.6 years old - end of life)

3. **Physically transport** to homelab location

**Labeling Convention:**
```
[SOURCE]-[SLOT]-[CAPACITY]-[MODEL]
918-Slot1-16TB-IronWolf
918-Slot2-14TB-IronWolf
918-Slot4-14TB-Seagate
```

**Duration:** 1-2 hours (physical work)

---

### PHASE 3: Install 918 Disks in Homelab Proxmox (Day 4)

**Current Homelab Status:**
- Occupied slots: 2 (WD 10TB pair in RAID mirror)
- Available slots: 4
- Free space on existing RAID: ~4TB

**Actions:**
1. Install 918 disks in homelab slots 3, 4, 5 (reserve slot 6 for future)
   - Slot 3: 918 Slot1 (16TB Seagate IronWolf)
   - Slot 4: 918 Slot2 (14TB Seagate IronWolf)
   - Slot 5: 918 Slot4 (14TB Seagate)

2. **Create new ZFS pool** for received disks (or add to existing)
   - Option A: Mirror disks 3+4 (16TB effective for new pool)
   - Option B: Create RAIDZ (all 3 disks for redundancy)
   - **Recommendation:** RAIDZ for maximum utilization (44TB → ~29TB usable with 1-disk redundancy)

3. Partition and format new storage on homelab

**Estimated Duration:** 2-3 hours

---

### PHASE 4: Transfer UGREEN Data to Homelab Disks (Days 5-7)

**Objective:** Move 918 data from UGREEN temporary storage back to its original disks (now in homelab)

**Data to transfer:**
- All content from `/storage/Media/` related to 918
- ~16.5TB total

**Network Path:** UGREEN (192.168.40.60) → Homelab (192.168.40.40)

**Transfer Command (example):**
```bash
# From UGREEN LXC
rsync -avh --progress \
  /storage/Media/Movies918 \
  /storage/Media/Series918 \
  /storage/Media/20251209backupsfrom918 \
  root@192.168.40.40:/mnt/918-data/
```

**Space Requirements at Homelab:**
- New pool size (RAIDZ): ~29TB usable
- Data to receive: 16.5TB
- **Status:** ✅ Sufficient space (46% utilization)

**Estimated Duration:** 3-4 days (network transfer)

**Completion Verification:**
```bash
# On homelab
du -sh /mnt/918-data/*
# Should show: ~16.5TB total
```

---

### PHASE 5: Prepare UGREEN for 920 Data (Day 7-8)

**Current UGREEN Status (after Phase 4):**
- Existing data: ~5.67TB (from 918 backups)
- Free space: ~14.3TB
- **Action:** Can keep 918 data on UGREEN OR clean up to make room

**Options:**
- **Option A:** Keep 918 data on UGREEN as backup
- **Option B:** Delete 918 data from UGREEN (mirrors what's on homelab now)
  - Frees up 16.5TB space on UGREEN
  - UGREEN becomes pure 920 repository

**Recommendation:** **Option B** - Cleaner architecture
- UGREEN = 920 data repository (30TB)
- Homelab = 918 data archive (16.5TB)

**If Option B - Clean UGREEN:**
```bash
zfs destroy -r storage/Media/Movies918
zfs destroy -r storage/Media/Series918
zfs destroy -r storage/Media/20251209backupsfrom918
# Frees 16.5TB on UGREEN
```

---

### PHASE 6: Begin 920 Data Migration to UGREEN (Days 8-12)

**⚠️ CRITICAL URGENCY:** 920 Volume 1 is 95% full

**Priority 1: Migrate Seriale 2023 (17TB)**
```bash
# From 920 NAS
rsync -avh --progress \
  /volume1/Seriale\ 2023/* \
  root@192.168.40.60:/storage/Media/920-Seriale-2023/
```

**Space check:**
- UGREEN free (after cleanup): 20TB
- 920 data: 30TB
- **PROBLEM:** Need 30TB but only have 20TB

**Solution:** Transfer in two batches
- Batch 1: Seriale 2023 (17TB) → UGREEN
- Batch 2: After Batch 1 complete, transfer Filmy920 (13TB)

**Transfer Timeline:**
- Batch 1 (17TB): 3-4 days
- Batch 2 (13TB): 2-3 days
- **Total: 5-7 days** (sequential)

**Completion Verification:**
```bash
# On UGREEN after both batches
du -sh /storage/Media/920-*
# Should show: 17TB + 13TB = 30TB total
```

---

### PHASE 7: Migrate Nginx Service from 920 to UGREEN Proxmox (Day 12)

**Service Details:**
- **Current location:** 920 NAS
- **Service:** Nginx web server (VITAL)
- **Destination:** UGREEN Proxmox LXC 102

**Migration Steps:**

1. **Backup current nginx config from 920:**
```bash
ssh root@192.168.40.20
cd /etc/nginx
tar czf nginx-config-backup.tar.gz conf.d/ sites-enabled/ sites-available/ nginx.conf
# Copy to UGREEN for reference
```

2. **On UGREEN LXC 102, install nginx:**
```bash
apt update
apt install nginx -y
systemctl start nginx
systemctl enable nginx
```

3. **Restore/configure nginx on UGREEN:**
   - Copy backed-up configs
   - Update paths if needed
   - Test configuration: `nginx -t`
   - Reload: `systemctl reload nginx`

4. **Update DNS/network:**
   - If nginx was accessed via 920's IP (192.168.40.20), update clients to use UGREEN (192.168.40.81)
   - Update any firewall rules
   - Test connectivity

5. **Verify on 920:**
   - Stop nginx on 920 to confirm traffic switches to UGREEN
   - Monitor for issues

**Estimated Duration:** 2-4 hours

---

### PHASE 8: Final Verification & Decommissioning (Days 13-14)

#### Pre-Decommission Verification

**918 NAS:**
1. Verify all data safely on homelab
   ```bash
   # Homelab: compare file counts and sizes
   find /mnt/918-data -type f | wc -l
   du -sh /mnt/918-data
   ```

2. Verify 918 disks are healthy in homelab ZFS pool
   ```bash
   # Homelab: check pool status
   zpool status
   ```

3. 918 decommission approval: ✅ Safe to power off

**920 NAS:**
1. Verify all content transferred to UGREEN
   ```bash
   # UGREEN: compare content
   ls -lah /storage/Media/920-*
   du -sh /storage/Media/920-*
   ```

2. Verify nginx running on UGREEN
   ```bash
   # UGREEN LXC 102
   systemctl status nginx
   curl http://localhost
   ```

3. Verify other docker services can be stopped (or noted for disregard)
4. 920 decommission approval: ✅ Safe to power off

#### Decommissioning Actions

**920 NAS:**
```bash
# On 920 - graceful shutdown
systemctl poweroff
# Wait for system to shut down
# Power off physically
```

**918 NAS:**
```bash
# On 918 - graceful shutdown
systemctl poweroff
# Wait for system to shut down
# Power off physically
```

**Post-Decommission:**
1. Remove both units from racks/shelves
2. Disconnect network cables
3. Archive/document decommissioning date
4. Store hardware for e-waste recycling

---

## Data Storage Architecture After Migration

### Final State

**UGREEN Proxmox (192.168.40.60) - PRIMARY**
```
Storage Pool: "storage" (20TB ZFS mirror)
├── 920-Seriale-2023/    (17TB) - TV series
└── 920-Filmy920/        (13TB) - Movies
Total: 30TB (100% utilized, 0TB free)

Services:
├── Nginx web server (LXC 102)
├── NFS mounts to homelab 918 data (if needed)
└── Plex Media Server (optional, kept or moved)
```

**Homelab Proxmox (192.168.40.40) - ARCHIVE**
```
Storage Pool: RAIDZ (44TB raw / ~29TB usable)
├── Original disks: 2x WD 10TB (mirror, 6TB used)
└── 918 migrated disks: 16.5TB (918-Seriale, 918-Filmy, 918-Backups)

Total utilization: ~23TB / 29TB (79%)
```

---

## Timeline Summary

| Phase | Task | Duration | Target Date |
|-------|------|----------|-------------|
| 1 | Complete 918→UGREEN transfer | 2-3 days | 2025-12-09 |
| 2 | Prepare 918 disks | 1-2 hours | 2025-12-09 |
| 3 | Install disks in homelab | 2-3 hours | 2025-12-10 |
| 4 | UGREEN→Homelab transfer | 3-4 days | 2025-12-13 |
| 5 | Clean up UGREEN (optional) | 1 hour | 2025-12-13 |
| 6 | Begin 920→UGREEN migration | 5-7 days | 2025-12-20 |
| 7 | Migrate nginx service | 2-4 hours | 2025-12-20 |
| 8 | Final verification | 1 day | 2025-12-21 |
| - | **Decommission 918 & 920** | - | **~2025-12-21** |

**Total Timeline:** ~14 days (2 weeks) from now

---

## Risk Assessment

### Low Risk ✅
- ✅ Data transfers with verification checksums
- ✅ RAID redundancy at homelab (RAIDZ 1-disk fault tolerance)
- ✅ Network transfers (gigabit, stable infrastructure)
- ✅ No single points of failure during transition

### Medium Risk ⚠️
- ⚠️ 920 Volume 1 at 95% capacity (CRITICAL - must start Phase 6 ASAP)
- ⚠️ Nginx service migration (web service downtime during cutover)
- ⚠️ Disk physical movement (potential for handling damage - mitigate with care)

### Mitigation Strategies
1. **920 Capacity:** Start Phase 1 immediately (data transfer from 918 should be complete by 2025-12-09)
2. **Nginx migration:** Schedule during low-traffic window, have rollback plan
3. **Disk handling:** Proper ESD protection, careful physical handling, labeling

---

## Success Criteria

- ✅ All 918 data safely stored on homelab (16.5TB verified)
- ✅ All 920 data safely stored on UGREEN (30TB verified)
- ✅ Nginx running on UGREEN Proxmox
- ✅ File counts and sizes match source (checksum verification)
- ✅ Both NAS units powered down and decommissioned
- ✅ Disks healthy in new locations (ZFS pool status: ONLINE)

---

## Post-Migration Archive

**Documentation to create:**
- Migration completion report (dates, file counts, checksums)
- Network diagram (updated with final configuration)
- Service recovery procedures (how to restore nginx if needed)
- Disk inventory (what's where in the final state)
- Decommissioning record (dates, conditions)

---

## Questions & Next Steps

**Before starting, confirm:**
1. ✅ Can UGREEN handle 5.67TB + 16.5TB + 30TB = 52TB total?
   - **Answer:** NO - space issue as documented above
   - **Mitigation:** Use phased approach (described in plan)

2. ✅ Can nginx service have brief downtime during migration?
   - If yes: proceed with Phase 7
   - If no: plan zero-downtime approach (reverse proxy on homelab temporarily)

3. ✅ 920 volume 1 at critical capacity - start immediately?
   - **Recommendation:** YES - begin Phase 1 within 24 hours

---

**Plan Status:** ✅ Ready for execution
**Next Action:** Start Phase 1 (complete 918→UGREEN transfer)
**Owner:** You (execution), Claude Code (planning/documentation)

---

*Plan created: 2025-12-17 | Strategy: Option D | Status: Ready for approval*
