# mumble-selinux

Written for [Mumble's](https://www.mumble.com) murmur daemon which provides a VOIP server on CentOS7.

This policy is intended to work for murmur without the ice and dbus functionality. Just as a basic murmur daemon for users to connect to.

Please follow the [official murmur CentOS7 installation guide](https://wiki.mumble.info/wiki/Install_CentOS7) as this policy was written to conform to it.

### Untested:
* ice and dbus functionality
* Using MySQL as a database instead of sqlite

## Installation
```sh
# Clone the repo
git clone https://github.com/georou/mumble-selinux.git

# Compile the selinux module (see below)

# Install the SELinux policy module. Compile it before hand to ensure proper compatibility (see below)
semodule -i murmurd.pp

# Add murmur ports
semanage port -a -t murmurd_port_t -p tcp 64738
semanage port -a -t murmurd_port_t -p udp 64738

# Restore all the correct context labels
restorecon -RvF /usr/local/murmur
restorecon -vF /etc/murmur.ini
restorecon -RvF /var/run/murmur
restorecon -RvF /var/log/murmur
restorecon -RvF /var/lib/murmur

# Start murmurd
systemctl start murmur.service

# Ensure it's working in the proper confinement
ps -eZ | grep murmur
```

## How To Compile The Module Locally (Needed before installing)
Ensure you have the `selinux-policy-devel` package installed.
```sh
# Ensure you have the devel packages
yum install selinux-policy-devel setools-console
# Change to the directory containing the .if, .fc & .te files
cd mumble-selinux
make -f /usr/share/selinux/devel/Makefile murmurd.pp
semodule -i murmurd.pp
```

## Debugging and Troubleshooting

* If you're getting permission errors, uncomment permissive in the .te file and try again. Re-check logs for any issues. Or `semanage permissive -a murmurd_t`
* Easy way to add in allow rules is the below command, then copy or redirect into the .te module. Rebuild and re-install:
* Don't forget to actually look at what is suggested. audit2allow will most likely go for a coarse grained permission!

```sh
ausearch -m avc,user_avc,selinux_err -ts recent | audit2allow -R
```
If you get a could not open interface info [/var/lib/sepolgen/interface_info] error. 
Ensure policycoreutils-devel is installed and/or run: `sepolgen-ifgen`
