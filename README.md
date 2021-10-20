### This is a fork of krathalan's systemd-sanboxing repo. Be aware, the changes here are being made by a Linux newbie - me. I'm playing around with configurations, restarting affected services on Ubuntu 20.04 and looking through error logs to understand if sandboxing configs are breaking apps.

## systemd-sandboxing
Drop-in `.conf` files for sandboxing systemd services.

## Explanation of options
View the `guide.conf` file in the root of this repository for a short explanation of each option. Most comments come from `systemd.exec(5)` and `systemd.resource-control(5)` man pages. Most options come from all of the options that `systemd-analyze security` uses to score a service.

## Installation
`systemctl edit [service]` and paste the hardening.conf contents under `### Anything between here and the comment below will become the new contents of the file`

### cupsd
The cupsd overrides have only been tested with an HP printer on the local network. Printers from other manufacturers or printers connected via a different interface (e.g. USB, SAMBA, etc.) may not work. Patches accepted.

### dictd
The dictd overrides assume that you only want to access the dictd server from your local machine. The pid file is somewhat useless, so its default location is not writable with these hardening options. Here is an example `dictd.conf` snippet for these hardening options:

```
global {
    site site.info
    pid_file /dev/null
}

# who's allowed.  You might want to change this.
access {
  allow 127.0.0.*
}
```

This snippet makes dictd not create a pid file and only allow access from the localhost.

### services run as unprivileged user
Some sandboxing options, like those used for nginx, opendkim, and opendmarc, assume you are running the service as an unprivileged user. There is also a `user.conf` file in their directory, in addition to the regular `hardening.conf`.

See these links for more information:

https://wiki.archlinux.org/index.php/Nginx#Running_unprivileged_using_systemd

https://wiki.archlinux.org/index.php/OpenDKIM#Security

### other caveats

The `sshd` override only adds IPAccounting, no sandboxing.

The `systemd-logind` override only adds the service to the `proc` group, un-breaking the service when `/proc` is mounted with `hidepid=2,gid=proc`.

The `systemd-networkd` override only adds `apparmor.service` to the `After=` option, so that the service doesn't start before its AppArmor profile is loaded.y
