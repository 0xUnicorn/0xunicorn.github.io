---
layout: post
title:  "Free disk space on linux with systemd"
date:   2025-06-07
author: Unicorn
tags: Linux Systemd
---

Today I had to perform some operations on a linux VM with `100% disk usage`.
This is not always straight forward, so I needed a way to free up some space, before doing my operations.

```bash
Filesystem      Size  Used Avail Use% Mounted on
udev            1.9G     0  1.9G   0% /dev
tmpfs           392M   36M  357M  10% /run
/dev/vda1        14G   14G     0 100% /
tmpfs           2.0G     0  2.0G   0% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs           392M     0  392M   0% /run/user/1000
```

After I went through the usual like `apt autoclean && apt autoremove` or removing files from `/tmp` and `/home`
without freeing any useful space, I turned to systemd.

Logs will always take up some space on a linux server, and using `journalctl` it's straightforward to clean those logs.

## Rotate + delete and vacuum journals

Verify the storage usage by logs with the `--disk-usage` parameter.

>**--disk-usage**\
> Shows the current disk usage of all journal files. This shows
> the sum of the disk usage of all archived and active journal
> files.
>
> Added in version 190.

```bash
$ sudo journalctl --disk-usage
Archived and active journals take up 136.0M in the file system.
```

### Rotate

When rotating logs, the current logfile gets marked as archived and renamed, while a new blank log file
gets created.

>**--rotate**\
> Asks the journal daemon to rotate journal files. This call
> does not return until the rotation operation is complete.
> Journal file rotation has the effect that all currently active
> journal files are marked as archived and renamed, so that they
> are never written to in future. New (empty) journal files are
> then created in their place. This operation may be combined
> with --vacuum-size=, --vacuum-time= and --vacuum-file= into a
> single command, see above.
>
> Added in version 227.

```bash
$ sudo journalctl --rotate
(No output from this command)
```

### Delete and vacuum

When old log files have been archived, we can delete and vacuum them to free up disk space.

>**--vacuum-size**\
> removes the oldest archived journal files until
> the disk space they use falls below the specified size.
> Accepts the usual "K", "M", "G" and "T" suffixes (to the base
> of 1024).
>
> Note that running **--vacuum-size=** has only an indirect effect
> on the output shown by --disk-usage, as the latter includes
> active journal files, while the vacuuming operation only
> operates on archived journal files

```bash
$ sudo journalctl --vacuum-size=100M
Deleted archived journal /var/log/journal/2..6/system@1..d.journal (8.0M).
Deleted archived journal /var/log/journal/2..6/system@1..f.journal (8.0M).
Vacuuming done, freed 868.5M of archived journals from /var/log/journal/2..6.
Vacuuming done, freed 0B of archived journals from /run/log/journal.
```

## Verify logs

Before calling it a day, use the `--verify` option to validate the integrity of the leftover logs.

>**--verify**\
> Check the journal file for internal consistency. If the file
> has been generated with FSS enabled and the FSS verification
> key has been specified with --verify-key=, authenticity of the
> journal file is verified.
>
> Added in version 189

```bash
$ sudo journalctl --verify
PASS: /var/log/journal/2..6/user-1000@d..d.journal
PASS: /var/log/journal/2..6/system@2..1.journal
PASS: /var/log/journal/2..6/user-1000@d..1.journal
PASS: /var/log/journal/2..6/system@2..a.journal
PASS: /var/log/journal/2..6/user-1000.journal
```

## Conclusion

I now have enough free space, to not get errors when trying to start/stop services or run commands,
with the need for creating temporary files.

```bash
Filesystem      Size  Used Avail Use% Mounted on
udev            1.9G     0  1.9G   0% /dev
tmpfs           392M  1.2M  391M  10% /run
/dev/vda1        14G   13G  144M  99% /
tmpfs           2.0G     0  2.0G   0% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs           392M     0  392M   0% /run/user/1000
```

Journalctl man page quotes from:
[journalctl(1)](https://man7.org/linux/man-pages/man1/journalctl.1.html)
