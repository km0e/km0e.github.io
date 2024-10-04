+++
title = 'Systemd User Service'
date = 2024-10-03T22:53:13+08:00
draft = false
categories = ["Linux"]
tags = ["systemd"]
+++
# auto stop

`systemd`'s `user` type service will stop automatically after `ssh` session is disconnected. The solution is as follows:

```bash
sudo loginctl enable-linger $USER
```

# service not found

`user` type `systemd` service does not exist, the error is as follows:
```bash
Failed to connect to bus: $DBUS_SESSION_BUS_ADDRESS and $XDG_RUNTIME_DIR not defined (consider using --machine=<user>@.host --user to connect to bus of other user)
```

The solution is to modify the `systemd` configuration file `/etc/ssh/sshd_config` as follows:

```bash
...
UsePAM yes
...
```

Don't forget to restart the `sshd` service:

```bash
sudo systemctl restart sshd
```