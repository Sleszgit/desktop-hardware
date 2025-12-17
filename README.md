# Hardware Inventory

**Repository:** Complete hardware documentation for all systems (Desktop, NAS, Infrastructure)
**Owner:** sleszgit
**Created:** 12.12.2025
**Last Updated:** 17.12.2025

---

## Overview

This repository contains comprehensive hardware documentation, analysis, and troubleshooting guides for all systems including desktop computer (DESKTOP24) and NAS devices (918 NAS, second Synology NAS, UGREEN NAS).

---

## Contents

### `/desktop/`
Desktop PC (DESKTOP24) hardware documentation:

- **hardware-info.txt** - Windows systeminfo output (complete specs)
- **DESKTOP24-ANALYSIS.md** - Comprehensive hardware analysis
  - Full component specifications
  - Performance assessments
  - Thermal analysis
  - Upgrade path planning
  - Maintenance schedules
  - Troubleshooting guides
  - Value assessment

### `/nas/`
NAS devices hardware documentation:

- **918-NAS-ANALYSIS.md** - Synology DS918+ inventory and health status
  - Complete disk specifications
  - Power-on hours and health metrics
  - RAID configuration
  - Storage capacity breakdown
  - Migration planning data

---

## System Specifications Summary

**DESKTOP24** - High-End Gaming/Workstation Desktop

| Component | Specification |
|-----------|---------------|
| **CPU** | Intel Core i7-14700KF (20 cores, 28 threads, up to 5.6 GHz) |
| **Motherboard** | ASRock Z790 Steel Legend WiFi |
| **RAM** | 64GB DDR5-4800 Corsair (2x 32GB) |
| **Cooling** | Corsair H150i ELITE LCD XT (360mm AIO) |
| **GPU** | NVIDIA GeForce RTX 4070 Ti SUPER (16GB) |
| **Storage** | 8TB NVMe SSD (990 EVO Plus 4TB + 2x 980 PRO 2TB) |
| **BIOS** | v21.02 (October 2025) |

**System Rating:** ⭐⭐⭐⭐⭐ (5/5) - Flagship/Enthusiast tier

**Estimated Value:** €2,800-3,800

---

**918 NAS** - Synology DS918+ Network Attached Storage

| Component | Specification |
|-----------|---------------|
| **Model** | Synology DiskStation DS918+ |
| **OS** | DSM 7.2.2 (Build 72806, Nov 10 2025) |
| **CPU** | Intel Celeron J3455 @ 1.50GHz (4 cores) |
| **RAM** | 16GB |
| **Storage Disks** | 4x SATA drives (56TB raw capacity) |
| **Disk Details** | Slot 1: Seagate IronWolf PRO 16TB (14,116 hrs) |
| | Slot 2: Seagate IronWolf PRO 14TB (30,459 hrs) |
| | Slot 3: WD Red Pro 10TB (40,646 hrs) |
| | Slot 4: Seagate 14TB (14,124 hrs) |
| **Total Capacity** | 36TB across 3 volumes (25-44% used) |
| **Network** | 1Gbps Gigabit Ethernet |
| **Health Status** | ✅ All disks healthy - SMART passed |

**Health Assessment:** ⭐⭐⭐⭐ (4/5) - Functional, aging disk in slot 3 (4.6 yrs) candidate for retirement

---

## Related Repositories

- **Homelab Hardware:** https://github.com/Sleszgit/homelab-hardware
- **UGREEN Infrastructure:** (Coming soon)

---

## Maintenance Status

✅ **DESKTOP24 Analysis:** Complete
✅ **918 NAS Inventory:** Complete (17.12.2025)
⏳ **Second Synology NAS:** Pending data collection
⏳ **UGREEN NAS:** Pending data collection
⏳ **Migration Plan:** Awaiting complete inventory

---

## Last Updated

**Date:** 17.12.2025
**Updated By:** Claude Code (ugreen-ai-terminal)
**Status:** Active documentation - NAS inventory in progress
**Recent Changes:** Added 918 NAS disk health and power-on hours data

---

## License

Personal hardware documentation - All Rights Reserved
