---
layout: post
title:  "QEMU/KVM with ZFS storage"
date:   2025-06-10
author: Unicorn
tags: ZFS Homelab QEMU/KVM
---

Today I've been migrating my `zvol` to file based `qcow2` images. I'm sure a well tuned system would benefit from the use of `zvol` over `qcow2`,
and even though this older test from `Jim Salter (Sanoid)` [ZVOL vs QCOW2 with KVM](https://jrs-s.net/2018/03/13/zvol-vs-qcow2-with-kvm/) (2018),
states that many benefits can be gained from using `qcow2` images, newer tests reach the opposite result. My system isn't well tuned enough for me to actually care,
I prefer security, easy maintenance and stability through out all of my homelab.


## ZFS tuning

### SSD Datasets

| Property            | Recommended Value | Reason |
|---------------------|------------------|--------|
| **atime**          | `off`            | Avoid unnecessary metadata writes. |
| **recordsize**      | `64K`            | Matches qcow2 cluster size to reduce fragmentation. |
| **compression**     | `lz4`            | Fast and efficient, reduces storage footprint. |
| **sync**           | `disabled`        | SSDs handle sync efficiently, avoiding redundant forced writes. |
| **primarycache**    | `all`            | Keeps both metadata and data in RAM for fast access. |
| **secondarycache**  | `none`           | No need for L2ARC caching (SSD already fast). |
| **autotrim**       | `on`             | Allows ZFS to discard freed blocks (SSD optimization). |

```bash
zpool1 (ssd) : `zfs set recordsize=64K atime=off compression=lz4 sync=disabled primarycache=all zpool1`
zpool1 (ssd) : `zpool set autotrim=on zpool1`
```

### HDD Datasets

| Property            | Recommended Value | Reason |
|---------------------|------------------|--------|
| **atime**          | `off`            | Avoids excessive metadata writes. |
| **recordsize**      | `128K`           | HDDs prefer large sequential writes for better performance. |
| **compression**     | `lz4`            | Reduces storage footprint while keeping CPU overhead low. |
| **sync**           | `standard`        | Ensures proper write ordering without forcing every write. |
| **primarycache**    | `metadata`       | Prefetches metadata to speed up directory and file lookups. |
| **secondarycache**  | `none`           | L2ARC caching won’t help much unless the HDD pool is heavily bottlenecked. |

```bash
zpool2/zpool3 (hdd) : `zfs set recordsize=128K atime=off compression=lz4 sync=standard primarycache=metadata secondarycache=none zpool2`
zpool2/zpool3 (hdd) : `zpool set autotrim=off zpool1`
```
## Enable discard for VMs

```xml
<disk type='file' device='disk'>
  <driver name='qemu' type='qcow2' discard='unmap'/>
  <source file='/usr/local/lib/libvirt/images/zpool1/<vm-name>/<vm-name>.qcow2'/>
  <target dev='vda' bus='virtio'/>
  <address type='pci' domain='0x0000' bus='0x04' slot='0x00' function='0x0'/>
</disk>
```

`<driver name='qemu' type='qcow2' discard='unmap'/>`

### Migrate datasets

Standard `cp` moves data inefficiently through the file system layer, whereas `ZFS send/receive` is much faster as it only tranfers the used blocks and not the unnecessary overhead.

So the procedure is the following for each dataset with a `qcow2` VM disk:

```bash
zfs rename tank/dataset tank/dataset_old
zfs snapshot tank/dataset_old@migrate
zfs send tank/dataset_old@migrate | zfs receive tank/dataset
```

