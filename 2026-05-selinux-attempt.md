# System Change Log - SELinux Enable Attempt

**Date:** 2026-05 (security audit session)
**Changed by:** Claude-assisted, approved by user
**Outcome:** Did not work — SELinux remains disabled

## Problem

A security audit found SELinux was completely disabled, which is a critical security
gap. SELinux is a kernel-level mandatory access control system — even if an app gets
exploited, SELinux confines what damage it can do.

## What We Tried

Changed `/etc/selinux/config` from `disabled` to `permissive`:

```bash
sudo sed -i 's/SELINUX=disabled/SELINUX=permissive/' /etc/selinux/config
```

Permissive mode logs violations but does not block anything — the safe starting point
before switching to enforcing mode.

## What Happened

After two reboots, `sestatus` still showed SELinux as `disabled`. The config file
change had no effect.

**Root cause:** Nobara disables SELinux at the **kernel/grub level**, not just in
the config file. The kernel boots with `selinux=0` passed as a boot argument, which
overrides the config file entirely.

A `/.autorelabel` file was also created during the attempt (triggers filesystem
relabeling on next boot) but since SELinux never activated, the relabeling never ran.

## Current State

`/etc/selinux/config` — reverted back to `SELINUX=disabled` to keep it clean.
SELinux is fully disabled as it was originally.

## Why Nobara Disables SELinux

Nobara is a gaming/desktop-focused Fedora variant. SELinux in enforcing mode can
break games, Steam, Wine, and Lutris. The Nobara team disables it by default for
compatibility.

## How To Properly Enable It In The Future

Enabling SELinux on Nobara requires a grub kernel parameter change:

```bash
# Remove the selinux=0 kernel argument
sudo grubby --update-kernel ALL --remove-args selinux

# Set config to permissive first (safe — logs only, doesn't block)
sudo sed -i 's/SELINUX=disabled/SELINUX=permissive/' /etc/selinux/config

# Reboot — first boot will be slow (filesystem relabeling, may take 5-10 min)
sudo reboot
```

After reboot, verify:
```bash
sestatus  # should show: permissive
```

Once stable on permissive, switch to enforcing:
```bash
sudo setenforce 1
sudo sed -i 's/SELINUX=permissive/SELINUX=enforcing/' /etc/selinux/config
```

## How To Revert (if enabled in future)

```bash
sudo grubby --update-kernel ALL --args selinux=0
sudo sed -i 's/SELINUX=.*/SELINUX=disabled/' /etc/selinux/config
sudo reboot
```
