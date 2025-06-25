---
layout: post
title:  "ZFS snapshots gone wild"
date:   2025-06-25
author: Unicorn
tags: ZFS Homelab
---

For some years I've been using [Sanoid](https://github.com/jimsalterjrs/sanoid) as my snapshot management/replication tool.

My snapshot policy was a "default/recomended" and was used for some years. But recently my snapshots started to clog up,
and have a huge accumulation of historic blocks so it was very much in need of some love.

For that reason I've been fine tuning my system for the past week or so and this is my journey through snapshot tuning by `curing diseases and not symptoms`.

## Snapshots

I changed my sanoid snapshot policy for my two production zpools to this:

```toml
[template_production]
frequently = 0
hourly = 36
daily = 14
monthly = 0
yearly = 0
```

As I have become wiser and realised a long running snapshot policy isn't worth it. I replicate to a `cold storage` anyway,
which is my archiving with long running snapshots.

## Pfsense usage

Not many days after some of my datasets already started taking up large amount of data
in this case with `pfsense2` it was with `7.13G REFER`

```bash
NAME USED AVAIL REFER MOUNTPOINT
zpool1/libvirt/pfsense2@autosnap_2025-06-10_15:12:32_daily 2.24G - 6.87G -
zpool1/libvirt/pfsense2@autosnap_2025-06-11_00:00:10_daily 2.00G - 6.97G -
zpool1/libvirt/pfsense2@autosnap_2025-06-12_00:00:12_daily 2.90G - 7.14G -
zpool1/libvirt/pfsense2@autosnap_2025-06-13_00:00:06_daily 2.62G - 6.87G -
zpool1/libvirt/pfsense2@autosnap_2025-06-14_00:00:08_daily 2.85G - 6.93G -
zpool1/libvirt/pfsense2@autosnap_2025-06-15_00:00:09_daily 2.68G - 6.99G -
zpool1/libvirt/pfsense2@autosnap_2025-06-16_00:00:11_daily 2.97G - 7.06G -
zpool1/libvirt/pfsense2@autosnap_2025-06-17_00:00:05_daily 2.60G - 7.09G -
zpool1/libvirt/pfsense2@autosnap_2025-06-18_00:00:12_daily 2.95G - 7.18G -
zpool1/libvirt/pfsense2@autosnap_2025-06-19_00:00:05_daily 2.78G - 7.17G -
zpool1/libvirt/pfsense2@autosnap_2025-06-19_21:15:04_hourly 203M - 7.13G -
zpool1/libvirt/pfsense2@autosnap_2025-06-19_22:00:01_hourly 49.5M - 7.13G -
zpool1/libvirt/pfsense2@autosnap_2025-06-19_23:00:04_hourly 10.7M - 7.15G -
zpool1/libvirt/pfsense2@autosnap_2025-06-19_23:15:03_hourly 6.37M - 7.16G -
zpool1/libvirt/pfsense2@autosnap_2025-06-20_00:00:02_daily 0B - 7.18G -
```

In my effort to try and `cure the disease` I went through these troubleshooting steps.

### Local logging

Historic blocks reserved by ZFS is often caused by small continous writes like `logs`.
So the first thing I tried was going into the pfense WebUI `Status > System Logs > Settings > Disable writing log files to the local disk`.

This action had a small warning attached: `WARNING: This will also disable Login Protection!`, but I'm okay with that temporarily.

I performed this action the `21-06-2025`, and this is how the change affected the snapshots up to the `25-06-25` not including `hourly` snapshots.

```bash
NAME                        USED  AVAIL  REFER  MOUNTPOINT
zpool1/libvirt/pfsense2    53.0G   116G  7.45G  /mnt/zpool1/libvirt/pfsense2
```

```bash
NAME                                                                USED  AVAIL  REFER  MOUNTPOINT
zpool1/libvirt/pfsense2@autosnap_2025-06-12_00:00:12_daily         3.56G      -  7.14G  -
zpool1/libvirt/pfsense2@autosnap_2025-06-13_00:00:06_daily         2.62G      -  6.87G  -
zpool1/libvirt/pfsense2@autosnap_2025-06-14_00:00:08_daily         2.85G      -  6.93G  -
zpool1/libvirt/pfsense2@autosnap_2025-06-15_00:00:09_daily         2.68G      -  6.99G  -
zpool1/libvirt/pfsense2@autosnap_2025-06-16_00:00:11_daily         2.97G      -  7.06G  -
zpool1/libvirt/pfsense2@autosnap_2025-06-17_00:00:05_daily         2.60G      -  7.09G  -
zpool1/libvirt/pfsense2@autosnap_2025-06-18_00:00:12_daily         2.95G      -  7.18G  -
zpool1/libvirt/pfsense2@autosnap_2025-06-19_00:00:05_daily         2.85G      -  7.17G  -
zpool1/libvirt/pfsense2@autosnap_2025-06-20_00:00:02_daily         2.95G      -  7.18G  -
zpool1/libvirt/pfsense2@autosnap_2025-06-21_00:00:09_daily         2.58G      -  7.20G  -
zpool1/libvirt/pfsense2@autosnap_2025-06-22_00:00:08_daily         2.25G      -  7.40G  -
zpool1/libvirt/pfsense2@autosnap_2025-06-23_00:00:12_daily         2.14G      -  7.50G  -
zpool1/libvirt/pfsense2@autosnap_2025-06-23_23:00:04_hourly        41.5M      -  7.50G  -
zpool1/libvirt/pfsense2@autosnap_2025-06-23_23:15:01_hourly        3.79M      -  7.50G  -
zpool1/libvirt/pfsense2@autosnap_2025-06-24_00:00:01_daily            0B      -  7.50G  -
```

The big change can be observed from the day before the change:

> zpool1/libvirt/pfsense2@autosnap_2025-06-20_00:00:02_daily         2.95G      -  7.18G  -

and up to this newest daily snapshot:

> zpool1/libvirt/pfsense2@autosnap_2025-06-23_00:00:12_daily         2.14G      -  7.50G  -

The snapshot size has gone down from `2.95G` to `2.14G`, so around `800M` smaller snapshots.

Now when the accumulation of historic blocks has been resolved, I needed the USED size to go down aswell.

First I deleted the oldest snapshots up until the day where I disabled the `local logging` (21-06-2025) and
this is the result:

```bash
NAME                        USED  AVAIL  REFER  MOUNTPOINT
zpool1/libvirt/pfsense2    19.7G   150G  7.45G  /mnt/zpool1/libvirt/pfsense2
```

A difference of over `33G` freed up.

But the `REFER` size hasn't gone down, actually it's still __increasing__.

The real disease causing the large snapshots hasn't been resolved yet. But I now know where I should look.

### ZFS Autotrim

After turning down the amount of writes performed by the VM and cleaning up old heavy snapshots, the `REFER`
hasn't moved a tiny bit.

This is an indication of the host ZFS filesystem don't know if any of the old reserved blocks has been freed within the VM itself.
For the host ZFS to know if certain blocks are freed, the VM should have access to `TRIM` functionality. This is granted by
the `discard='unmap'` in the VM XML configuration for `QEMU/KVM`.

But since `PfSense` also uses `ZFS` we need to enable `Autotrim` for the zpool within the VM, before it actually starts telling the host what blocks are free.

```bash
[2.7.2-RELEASE][admin@pfsense2]/root: zpool get autotrim pfsense2
NAME      PROPERTY  VALUE     SOURCE
pfsense2  autotrim  off        local

[2.7.2-RELEASE][admin@pfsense2]/root: zpool set autotrim=on pfsense2

[2.7.2-RELEASE][admin@pfsense2]/root: zpool get autotrim pfsense2
NAME      PROPERTY  VALUE     SOURCE
pfsense2  autotrim  on        local
```

I issued a manual `zpool trim pfsense2` command for running a trim on the disk. The command doesn't give any output, but running `zpool status pfsense2`
showed the device was trimming.

```bash
[2.7.2-RELEASE][admin@pfsense2]/root: zpool status pfsense2
NAME        STATE     READ WRITE CKSUM
pfsense2    ONLINE       0     0     0
  vtbd0p3   ONLINE       0     0     0 (trimming)
```

### Result

After disk trimming within the VM was done, the following output had confirmed the real reason for my snapshots taking up this much space over time
indeed was caused by the lack of `TRIM` on the `ZFS` datastore.

```bash
NAME                        USED  AVAIL  REFER  MOUNTPOINT
zpool1/libvirt/pfsense2    19.7G   149G   981M  /mnt/zpool1/libvirt/pfsense2
```

The `REFER` has gone from `~7.5G` down to `~1G`, that's a remarkable change, and I'm very satisfied.

Next up is letting a couple of days pass before evaluating, and then re-enable `local logging` if everything is still fine.

