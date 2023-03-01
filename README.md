# fips-mode-user-space

This repository manages a script to test with the OpenSSL with Federal Information Processing Standard (FIPS) mode enabled or disabled.

## Motivation

The OpenSSL and OpenSSL language bindings such as [OpenSSL for Ruby](https://github.com/ruby/openssl) have the implementation in the FIPS mode. However, testing with the FIPS mode enabled is not easy, because it requires some changes including the kernel parameter `fips=1` or `0` and rebooting the system in an official way. Fortunately, OpenSSL only depends on the content of the kernel FIPS flag (`/proc/sys/crypto/fips_enabled`) in some settings of enabling the FIPS mode.

## Design philosophy

The design philosophy of the script is only to change the minimal setting and time. The change by the script is reset when rebooting OS respecting the system's default FIPS mode setting.

## How to use

Show the status of the kernel FIPS flag. (1: enabled, 0: disabled)

```bash
# fips-mode-user-space-setup status
```

Enable the kernel FIPS flag.

```bash
# fips-mode-user-space-setup enable
```

Disable the kernel FIPS flag.

```bash
# fips-mode-user-space-setup disable
```

## Use cases

The cases were tested by the following environment.

```
$ cat /etc/redhat-release
Red Hat Enterprise Linux release 9.1 (Plow)

$ uname -r
5.14.0-162.17.1.el9_1.x86_64

$ rpm -qf /usr/bin/fips-mode-setup
crypto-policies-scripts-20220815-1.git0fbe86f.el9.noarch
```

### Enable and disable the kernel FIPS flag

Check the current FIPS mode status.

```
# fips-mode-setup --check
Installation of FIPS modules is not completed.
FIPS mode is enabled.
Inconsistent state detected.
```

```
$ ./fips-mode-user-space-setup status
/proc/sys/crypto/fips_enabled: 1
```

Enable the kernel FIPS flag.

```
# ./fips-mode-user-space-setup enable
```

Check the current FIPS mode status is enabled.

```
$ ./fips-mode-user-space-setup status
/proc/sys/crypto/fips_enabled: 1
```

The command below prints an error message "Inconsistent state detected.". This is an expected behavior.

```
# fips-mode-setup --check
Installation of FIPS modules is not completed.
FIPS mode is enabled.
Inconsistent state detected.

# echo $?
1
```

Disable the kernel FIPS mode by the command below.

```
# ./fips-mode-user-space-setup disable
```

```
$ ./fips-mode-user-space-setup status
/proc/sys/crypto/fips_enabled: 0
```

Then the error message above is disappeared.

```
# fips-mode-setup --check
Installation of FIPS modules is not completed.
FIPS mode is disabled.

# echo $?
0
```

### Enable the kernel FIPS flag and reboot the system.

Check the current FIPS mode status.

```
# fips-mode-setup --check
Installation of FIPS modules is not completed.
FIPS mode is enabled.
Inconsistent state detected.
```

```
$ ./fips-mode-user-space-setup status
/proc/sys/crypto/fips_enabled: 1
```

Enable the kernel FIPS flag.

```
# ./fips-mode-user-space-setup enable
```

Check the current FIPS mode status is enabled.

```
$ ./fips-mode-user-space-setup status
/proc/sys/crypto/fips_enabled: 1
```

Reboot the system.

```
# reboot
```

The kernel FIPS mode is reset as system default.

```
$ ./fips-mode-user-space-setup status
/proc/sys/crypto/fips_enabled: 0
```

```
$ fips-mode-setup --check
Installation of FIPS modules is not completed.
FIPS mode is disabled.

$ echo $?
0
```

## Notes

* This script can be used for only testing purposes.
* The changes by running this script are temporary. It is not preserved between rebooting the system.
* You can disable the FIPS mode temporarily in the system with FIPS mode enabled.
* There are documents about FIPS mode in Red Hat Enterprise Linux and Ubuntu.[1][2]
* This script may cause an error "Inconsistent state detected." by the officially provided command `fips-mode-setup`[1] (see `man fips-mode-setup` in RHEL 9) script. When you see the error, please undo the status by `fips-mode-user-space-setup disable` or `fips-mode-user-space-setup enable`. See the section Use cases - Enable the kernel FIPS flag.
* Please be sure to use it at your own risk.

## References

* [1] [RHEL 9 - Security hardening - 3.4. Switching the system to FIPS mode](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html-single/security_hardening/index#switching-the-system-to-fips-mode_using-the-system-wide-cryptographic-policies)
* [2] [Ubuntu - Enabling FIPS with the ua tool](https://ubuntu.com/security/certifications/docs/fips-enablement)
