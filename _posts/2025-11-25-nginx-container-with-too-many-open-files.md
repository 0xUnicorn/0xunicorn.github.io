---
layout: post
title:  "Nginx container with too many open files"
date:   2025-11-22
author: Unicorn
tags:
- Nginx
- Linux
- Docker
---

The other day I started to notice **something** was behaving funky. I could not point out if it was my linux installation,
my browser or the network. But **something** wasn't right and I did not have the time to dig into it right then and there.
Yesterday I took time to care for my infrastructure and found this error, on a host running an internal DNS service behind a Nginx container.

```bash
6e4e2a7a68ce[2500279]: 2025/11/21 21:45:13 [alert] 22#22: *268159 socket() failed (24: Too many open files)
while connecting to upstream, udp client: <redacted>, server: 0.0.0.0:53, upstream: "172.18.0.3:53",
bytes from/to client:30/0, bytes from/to upstream:0/0
```

## What is causing this issue?

Searching around the web, I could see this is related to the process (container) not being able to open any more file descriptors,
which is used by Nginx when it establishes a connection, with either client or upstream server.

I once had an issue with Nginx exhausting its 512 default `worker_connections` so I bumped it to 16384,
which resolved the issue and I did not think any more of it.

Now I have to understand what this mean, and looking around I found a relation between me bumping `worker_connections`
and not being able to open any more file descriptors. This is what I learned about the different parts involved.

- **worker_connections** - The amount of simultaneous conections for a `worker_processes` (Max clients = worker_connections * worker_processes).
- **soft limit** - `ulimit -n` is the enforced limit for process which on linux is `1024` as default.
- **hard limit** - `ulimit -Hn` is the maximum ceiling the OS allow for process which on linux is around `100000` as default.
- **/proc/sys/fs/file-max** - This is the kernel wide limit of `9223372036854775807` which is too big to be an issue.

My Nginx conf has `worker_processes auto;` set, which will try to autodetect the amount of CPU's and setting a matching value.
So with my 2 CPU's the Max client will be `16384 * 2 = 32768` and each of these connections requires at least one and maybe more file descriptors,
meaning a soft limit of `1024` is not enough to serve my clients.

## How to change ulimit for docker?

It would be possible to raise `ulimit` for all of the root user by running `ulimit -n <value>`, but this is not what I want.
When running docker with `systemctl` the default `ulimit` gets set by `LimitNOFILE` and `LimitNOFILESoft`:

```bash
~$ sudo systemctl show docker | grep -i nofile
LimitNOFILE=524288
LimitNOFILESoft=1024
```

but this can be overridden when starting a container with `docker run` or when using `docker compose`, by setting a config value of:

```yaml
service:
  image: image
  ulimits:
    nofile: 8192
```

Remember to take down the container for applying the change and verify it was applied by either running `ulimit -n` inside the container,
or by using:

```bash
~$ docker inspect | grep -A5 Ulimits
        "Ulimits": [
            {
                "Name": "nofile",
                "Hard": 8096,
                "Soft": 8096
            }
```

## Conclusion

A file descriptor soft limit of `1024`, is not enough for serving this amount of connections and `16384` is not the right setting for my `worker_connections`.

So I have chosen to bump my VM's CPU from 2 to 4 cores, and set the `worker_connections` to `2048`, so I now have
- `4 * 2048 = 8192`

With a matching nofile ulimit set to `8192` for my nginx docker container.

This should allow for plenty more DNS clients in the future, without me getting bothered by this issue again.

It also made me crave for some metrics of how many requests my different nginx containers actually receives, so in the near future,
I want to get this metric into my Prometheus/Grafana stack with a pretty dashboard.

