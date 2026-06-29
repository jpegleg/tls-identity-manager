![cdlogo](https://carefuldata.com/images/cdlogo.png)

# TLS identity manager

This program is a controller that reacts to certificate expiration date thresholds.

There is a `policy.yaml` file that controls when to react and how to react.

```
daemon:
  interval_seconds: 5400
  jitter_max_hours: 23
  connect_timeout_seconds: 29

endpoints:
  - host: carefuldata.com
    port: 443
    thresholds:
      - days_before_expiry: 13
        commands:
          - "/root/certbot_cd.sh"

      - days_before_expiry: 10
        commands:
          - "bash /root/cd_wrap"

```

The program can be run in the foreground, or installed as a system daemon.

The expected use is a system daemon that is installed via systemd or (rc init script, etc).
The tls-identity-manager service can then use long periods of jitter which is helpful for avoiding
burst traffic patterns and scrambles trackers that try to detect cert renewal date proximity for malicious purposes.

When run as root, the commands are run as root. It is root command injection as a service, so use with caution and protect the config file.
TLS identity manager can also be run as a lower privileges, that can work fine too.

Here is an example of running the program in debug mode in a terminal session:

```
$ tls-identity-manager /etc/tim/policy.yaml -d
[2026-06-28T05:11:43Z] - tls-identity-manager - "bastion1A" - INFO  - tls-renewal-manager v1.0.0 started  policy=policy.yaml  interval=333s  jitter_max=0.00h  connect_timeout=33s
[2026-06-28T05:11:43Z] - tls-identity-manager - "bastion1A" - INFO  - watching 'example.com:443' -> 4 threshold(s)
[2026-06-28T05:11:43Z] - tls-identity-manager - "bastion1A" - INFO  - watching 'another.domain.local:443' -> 3 threshold(s)
[2026-06-28T05:11:43Z] - tls-identity-manager - "bastion1A" - INFO  - watching 'something.localnet:443' -> 3 threshold(s)
[2026-06-28T05:11:44Z] - tls-identity-manager - "bastion1A" - DEBUG - [example.com:443] 56 day(s) until expiry
[2026-06-28T05:11:44Z] - tls-identity-manager - "bastion1A" - WARN  - [example.com:443] 56 day(s) remaining -> 100d threshold reached -> scheduling 1 reaction(s) after 0.00h jitter
[2026-06-28T05:11:44Z] - tls-identity-manager - "bastion1A" - INFO  - [example.com:443] running reaction: /usr/local/bin/renew example.com
[2026-06-28T05:11:44Z] - tls-identity-manager - "bastion1A" - DEBUG - [example.com:443] command ok (exit 0)
[2026-06-28T05:11:44Z] - tls-identity-manager - "bastion1A" - DEBUG - [another.domain.local:443] 63 day(s) until expiry
[2026-06-28T05:11:44Z] - tls-identity-manager - "bastion1A" - DEBUG - [something.localnet:443] 46 day(s) until expiry
[2026-06-28T05:11:44Z] - tls-identity-manager - "bastion1A" - DEBUG - cycle completed 1260ms -> sleeping 331740ms

```

TLS identity manager has three modes, `d` for debug mode, `-w` for warn mode, and `-q` for quiet mode.

Each theshold can have up to 255 commands. Each command is executed with `sh -c` so the system must have an `sh` shell installation to use TLS identity manager.

### installing

There is a required dependency of system openssl pacakges. The choice is to use the same system openssl/libressl/etc. If your install fails, this is the likely item
that is missing. Install the openssl sys libraries for your distro and then try to install again.

Example for debian:

```
sudo apt install librust-openssl-sys-dev
```

TLS identity manager can be compiled from source code such as with cargo, either with cargo install or clone the source code and building.

The binary can be downloaded from release binaries on GitHub.

```
cargo install tls-identity-manager
```

Write your own policy.yaml and scripts to execute the actions desired.

### openbsd version

See the [OpenBSD fork](https://github.com/jpegleg/paludification_toad/tree/main/timbsd) for work on Pledge and Unveil adoption.

### related software

See [linux disk space manger](https://github.com/jpegleg/linux-disk-space-manager) for a similar controller but for disk space.

