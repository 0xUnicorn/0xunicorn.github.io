---
layout: post
title:  "Expanding ZFS mirror from 8TB to 18TB"
date:   2025-11-22
author: Unicorn
tags:
- Hardware
- ZFS
---


```bash
~$ zpool status
  pool: zpool1
 state: ONLINE
  scan: scrub repaired 0B in 03:27:23 with 0 errors on Sun Nov  9 03:51:31 2025
config:

	NAME                                 STATE     READ WRITE CKSUM
	zpool1                               ONLINE       0     0     0
	  mirror-0                           ONLINE       0     0     0
	    ata-ST8000DM004-2U9188_ZR15AR2N  ONLINE       0     0     0
	    ata-ST8000DM004-2U9188_ZR15BCEG  ONLINE       0     0     0

errors: No known data errors
```

```bash
~$ zpool status
  pool: zpool1
 state: DEGRADED
status: One or more devices has been removed by the administrator.
	Sufficient replicas exist for the pool to continue functioning in a
	degraded state.
action: Online the device using zpool online' or replace the device with
	'zpool replace'.
config:

	NAME                                  STATE     READ WRITE CKSUM
	zpool1                                DEGRADED     0     0     0
	  mirror-0                            DEGRADED     0     0     0
	    ata-ST8000DM004-2U9188_ZR15AR2N   REMOVED      0     0     0
	    ata-ST8000DM004-2U9188_ZR15BCEG   ONLINE       0     0     0

errors: No known data errors
```

```bash
~$ zpool status
  pool: zpool1
 state: ONLINE
status: Some supported and requested features are not enabled on the pool.
	The pool can still be used, but some features are unavailable.
action: Enable all features using 'zpool upgrade'. Once this is done,
	the pool may no longer be accessible by software that does not support
	the features. See zpool-features(7) for details.
  scan: resilvered 4.73T in 11:00:39 with 0 errors on Sat Nov 22 04:46:15 2025
config:

	NAME                                  STATE     READ WRITE CKSUM
	zpool1                                ONLINE       0     0     0
	  mirror-0                            ONLINE       0     0     0
	    ata-ST18000NE000-3G6101_ZVTKGJT0  ONLINE       0     0     0
	    ata-ST8000DM004-2U9188_ZR15BCEG   ONLINE       0     0     0

errors: No known data errors
```

```bash
~$ zpool status
  pool: zpool1
 state: DEGRADED
status: One or more devices has been removed by the administrator.
	Sufficient replicas exist for the pool to continue functioning in a
	degraded state.
action: Online the device using zpool online' or replace the device with
	'zpool replace'.
  scan: resilvered 4.73T in 11:00:39 with 0 errors on Sat Nov 22 04:46:15 2025
config:

	NAME                                  STATE     READ WRITE CKSUM
	zpool1                                DEGRADED     0     0     0
	  mirror-0                            DEGRADED     0     0     0
	    ata-ST18000NE000-3G6101_ZVTKGJT0  ONLINE       0     0     0
	    ata-ST8000DM004-2U9188_ZR15BCEG   REMOVED      0     0     0

errors: No known data errors
```

```bash
~# zpool replace zpool1 ata-ST8000DM004-2U9188_ZR15BCEG /dev/disk/by-id/ata-ST18000NE000-3G6101_ZVTKGFF5
~# zpool status
  pool: zpool1
 state: DEGRADED
status: One or more devices is currently being resilvered.  The pool will
	continue to function, possibly in a degraded state.
action: Wait for the resilver to complete.
  scan: resilver in progress since Sat Nov 22 09:57:56 2025
	169G / 4.72T scanned at 15.3G/s, 0B / 4.72T issued
	0B resilvered, 0.00% done, no estimated completion time
config:

	NAME                                    STATE     READ WRITE CKSUM
	zpool1                                  DEGRADED     0     0     0
	  mirror-0                              DEGRADED     0     0     0
	    ata-ST18000NE000-3G6101_ZVTKGJT0    ONLINE       0     0     0
	    replacing-1                         DEGRADED     0     0     0
	      ata-ST8000DM004-2U9188_ZR15BCEG   REMOVED      0     0     0
	      ata-ST18000NE000-3G6101_ZVTKGFF5  ONLINE       0     0     0

errors: No known data errors
```

```bash
~# zpool status
  pool: zpool1
 state: DEGRADED
status: One or more devices is currently being resilvered.  The pool will
	continue to function, possibly in a degraded state.
action: Wait for the resilver to complete.
  scan: resilver in progress since Sat Nov 22 09:57:56 2025
	1.71T / 4.72T scanned at 272M/s, 1.24T / 4.72T issued at 196M/s
	1.24T resilvered, 26.15% done, 05:10:58 to go
config:

	NAME                                    STATE     READ WRITE CKSUM
	zpool1                                  DEGRADED     0     0     0
	  mirror-0                              DEGRADED     0     0     0
	    ata-ST18000NE000-3G6101_ZVTKGJT0    ONLINE       0     0     0
	    replacing-1                         DEGRADED     0     0     0
	      ata-ST8000DM004-2U9188_ZR15BCEG   REMOVED      0     0     0
	      ata-ST18000NE000-3G6101_ZVTKGFF5  ONLINE       0     0     0  (resilvering)

errors: No known data errors
```
