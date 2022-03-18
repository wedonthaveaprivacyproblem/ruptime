![ruptime](ruptime.png?raw=true "ruptime")

poor man’s ruptime

Historically it is using broadcast udp/513 [3] in a network.
So this is a test how many machines it can take.

## Never heard of ruptime, what does it look like?
```
$ ruptime
fish.ocean.net          up    4+21:27,  0 users,  load 0.22, 0.25, 0.25
tuna.ocean.net          up    4+21:27,  0 users,  load 0.20, 0.25, 0.25
dolphin.ocean.net       up   15+05:57,  0 users,  load 0.04, 0.08, 0.07
```

## Why would I want this?
- it's simple [4]
- monitoring systems have no or not very useful CLI tools
- you don't want to manually keep a list of hosts
- you want to see what hosts are down
- you want to see what hosts are not idle
- you want to run something on all running hosts with `parallel`

## Plan

- allow to monitor custom variables
- web presentation/views
- replace `mcrypt` with `openssl`

## Configuration
The defaults for rwhod/ruptime is downtime after 11' (11\*60 seconds) [1] (ISDOWN), status messages are originally generated approximately every 3' (AL_INTERVAL) [2].
```
SERVER=wedonthaveaprivacyproblem.com
PORT=51300
HOSTNAMECMD='hostname -f'
```

Create a key for the encryption with `mcrypt`. You will need this on server and client for symmetric encryption.
```
COLUMNS=160 dd if=/dev/urandom bs=1 count=60 2>/dev/null > /etc/ruptime/ruptime.key
```

Create a local user to run the daemon.
```
adduser --disabled-password --quiet --system --home /var/spool/ruptime --gecos "ruptime daemon" --group ruptime
```

Running the daemon.
```
daemon --user=ruptime:ruptime mini-inetd 51300 /usr/sbin/ruptimed
```

## Requirements
- Client: `nc` `mcrypt`
- Server: `nc` `xz` `tcputils` `daemon` `mcrypt`
- Optionals: `pen` `trickle` `timeout`

## Supported Systems
- macOS
- Linux
- FreeBSD
- Windows if someone implements uptime.exe (https://www.windowscentral.com/how-check-your-computer-uptime-windows-10#check_pc_uptime_cmd)

## Starting it
- FreeBSD: rc.d
- Linux: daemon, init.d, cron @reboot, systemd [5]
- macOS: https://medium.com/swlh/how-to-use-launchd-to-run-services-in-macos-b972ed1e352
- Windows (not sure if they still have `net start`, haven't seen it since NT 4)

- without systemd
```
crontab -l
*/3 * * * * /usr/bin/ruptime -u
```

- with systemd

```
/etc/systemd/system/ruptime.service
[Unit]
Description=ruptime
After=network.target
Documentation=https://github.com/alexmyczko/ruptime/

[Service]
Type=oneshot
ExecStart=/usr/bin/ruptime -u

[Install]
WantedBy=multi-user.target
```

```
/etc/systemd/system/ruptime.timer
[Unit]
Description=ruptime

[Timer]
OnBootSec=3m

[Install]
WantedBy=basic.target
```

## References
[1] https://sources.debian.org/src/netkit-rwho/0.17-14/ruptime/ruptime.c/

[2] https://sources.debian.org/src/netkit-rwho/0.17-14/rwhod/rwhod.c/

[3] https://manpages.debian.org/unstable/manpages/services.5.en.html

[4] https://www.gkogan.co/blog/simple-systems/

[5] https://kill-9.xyz/harmful/software/systemd
