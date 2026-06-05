# System Change Log - Lid Suspend Fix

**Date:** 2026-05-05
**Changed by:** Codex-assisted, approved by user

## Problem

The laptop became very hot after the lid was closed and it was carried in a bag.
System logs showed that the lid-close event was detected, but the laptop did not
successfully suspend. KDE PowerDevil tried to handle sleep, but suspend/hibernate
requests were blocked by a Polkit `multiple sessions` authorization issue. A later
systemd `suspend-then-hibernate` attempt also failed with `Device or resource busy`.

## Why This Was Done

The goal of these changes was to make lid-close suspend more reliable and prevent
the laptop from staying awake in a closed bag.

## Files Changed

### `/etc/systemd/logind.conf.d/90-safe-lid-suspend.conf`

This is a `systemd-logind` drop-in configuration file. It tells the system to
suspend when the lid is closed, including when on charger or docked/external-monitor
cases.

```ini
[Login]
HandleLidSwitch=suspend
HandleLidSwitchExternalPower=suspend
HandleLidSwitchDocked=suspend
LidSwitchIgnoreInhibited=yes
```

### `/etc/polkit-1/rules.d/49-<username>-suspend.rules`

This is a Polkit authorization rule. It allows the active user `<username>`, who is
in the `wheel` group, to suspend or hibernate even when systemd sees multiple
sessions. This addresses the `suspend-multiple-sessions` and
`hibernate-multiple-sessions` authorization failures seen in the logs.

```javascript
polkit.addRule(function(action, subject) {
    if ((action.id == "org.freedesktop.login1.suspend" ||
         action.id == "org.freedesktop.login1.suspend-multiple-sessions" ||
         action.id == "org.freedesktop.login1.hibernate" ||
         action.id == "org.freedesktop.login1.hibernate-multiple-sessions") &&
        subject.user == "<username>" &&
        subject.active &&
        subject.isInGroup("wheel")) {
        return polkit.Result.YES;
    }
});
```

No existing system configuration files were directly edited. These were added as
separate drop-in/rules files so the change is easier to inspect and undo later.

## Commands Used

Created the directories and files with `sudo`.

Reloaded `systemd-logind` after the change:

```bash
sudo systemctl reload systemd-logind
```

## How To Verify Later

Show effective lid-close configuration:

```bash
systemd-analyze cat-config systemd/logind.conf
```

Show current sleep blockers:

```bash
systemd-inhibit --list
```

Check suspend-related logs:

```bash
journalctl -b -g "lid|suspend|sleep|hibernate|polkit|PowerDevil"
```

## How To Undo

Remove the two added files:

```bash
sudo rm /etc/systemd/logind.conf.d/90-safe-lid-suspend.conf
sudo rm /etc/polkit-1/rules.d/49-<username>-suspend.rules
sudo systemctl reload systemd-logind
```

## Habit For Future Changes

For future system changes, keep a small note like this with:

- Date
- What problem happened
- What files were changed or added
- Why the change was made
- How to verify it
- How to undo it
