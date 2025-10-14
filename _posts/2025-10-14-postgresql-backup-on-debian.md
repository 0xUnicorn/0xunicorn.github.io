---
layout: post
title:  "Postgresql backup on Debian"
date:   2025-10-14
author: Unicorn
tags: Postgresql, Systemd
---

When running `Debian` as an OS, the default `postgresql-common` package ships with a tool called `pg_backupcluster` with the following description

```bash
pg_backupcluster - simple pg_basebackup and pg_dump front-end

pg_backupcluster provides a simple interface to create PostgreSQL cluster backups using pg_basebackup(1) and pg_dump(1).

To ease integration with systemd operation, the alternative syntax "pg_basebackup version-cluster action" is also supported.
```

and for scheduling backup using systemd, the needed service and timer units is also provided, ready to be enabled.

**/lib/systemd/system/pg_dump@.service**
```bash
[Unit]
Description=Dump of PostgreSQL Cluster %i
AssertPathExists=/etc/postgresql/%I/postgresql.conf
Wants=postgresql@%i.service
After=postgresql@%i.service
RequiresMountsFor=/var/backups/postgresql

[Service]
Type=oneshot
User=postgres
Environment="KEEP=3"
ExecStartPre=+/usr/bin/pg_backupcluster %i createdirectory
ExecStart=/usr/bin/pg_backupcluster %i dump
ExecStart=/usr/bin/pg_backupcluster %i expiredumps $KEEP
```

**/lib/systemd/system/pg_dump@.timer**
```bash
[Unit]
Description=Weekly Dump of PostgreSQL Cluster %i
AssertPathExists=/etc/postgresql/%I/postgresql.conf

[Timer]
OnCalendar=weekly
RandomizedDelaySec=1h
FixedRandomDelay=true

[Install]
# when enabled, start along with postgresql@%i
WantedBy=postgresql@%i.service
```

## Enable automated pg_dump backups

For enabling the backup timer unit, you need to know the version and the cluster name.
The easisest way to obtain this is looking at the config folder in `/etc/postgresql/<version>/<cluster-name>`

```bash
systemctl --now enable pg_dump@<version>-<cluster-name>
Created symlink /etc/systemd/system/postgresql@15-main.service.wants/pg_dump@15-main.timer → /lib/systemd/system/pg_dump@.timer.
```

This automated backup will currently run `once a week` and keep the last `3 backups` at the location `/var/backups/postgresql/<version>-<cluster-name>`.

### Custom schedule

A weekly backup doesn't really cut it, so I added a custom timer unit with a daily schedule instead

```bash
[Unit]
Description=Custom Daily Dump of PostgreSQL Cluster %i
AssertPathExists=/etc/postgresql/%I/postgresql.conf

[Timer]
OnCalendar=daily
RandomizedDelaySec=1h
FixedRandomDelay=true

[Install]
# when enabled, start along with postgresql@%i
WantedBy=postgresql@%i.service
```

I then saved it as `/etc/systemd/system/pg_dump@<version-<cluster-name>.timer` and enabled it.

## Fetching backups from remote host

I added a cronjob using rsync for copying the backup files from the postgres servers back to the backup server

```bash
5 2 * * * rsync -az backup_svc@[postgres_server]:/var/backups/postgresql/ /home/backup_svc/backup/[postgres_server]/
```

