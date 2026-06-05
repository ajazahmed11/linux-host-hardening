# System Change Log - dnf-automatic (Auto Security Updates)

**Date:** 2026-05 (security audit session → disabled same session)
**Changed by:** Claude-assisted, approved by user
**Outcome:** Enabled then disabled in the same session — currently OFF

## Problem

Automatic security updates were disabled — the system only received patches when
manually running `dnf update`. This meant security vulnerabilities could go unpatched
for days or weeks.

## What We Did

**Enabled** automatic security updates:

```bash
sudo systemctl enable --now dnf5-automatic.timer
```

This runs `dnf-automatic` daily in the background, automatically downloading and
applying security-only updates.

**Then disabled it** the same session due to concern it was causing Dolphin/taskbar
freezes (background disk I/O during a download spike):

```bash
sudo systemctl disable --now dnf5-automatic.timer
```

Investigation later showed dnf-automatic was not actually causing the freeze (the
freeze was caused by rclone HTTP timeouts — see rclone log). But it was left disabled.

## Current State

`dnf5-automatic.timer` — **disabled and inactive**

Updates are now done manually. Recommended habit: run once a week.

```bash
# Security updates only
sudo dnf upgrade --security

# All updates
sudo dnf upgrade
```

## How To Re-enable

```bash
sudo systemctl enable --now dnf5-automatic.timer
```

Verify it's scheduled:
```bash
systemctl status dnf5-automatic.timer
```

## How To Disable Again

```bash
sudo systemctl disable --now dnf5-automatic.timer
```
