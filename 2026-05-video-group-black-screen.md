# System Change Log - Video Group & Black Screen Fix Attempt

**Date:** 2026-05 (across multiple sessions)
**Changed by:** Claude-assisted, approved by user
**Outcome:** Partial fix — video group added (permanent), black screen still occurs

## Problem

Black screen after login on every boot. The workaround was closing and reopening the
laptop lid to trigger a display refresh. This was traced to KWin (the KDE window
manager) failing to access GPU hardware.

## Root Cause Investigation

Logs showed KWin failing with:

```
kwin_wayland: Failed to open drm node: /dev/dri/card1 — Permission denied
```

`/dev/dri/card1` is owned by the `video` group. The user `<username>` was not in the
`video` group, so KWin silently failed to open the display device.

A second deeper cause was also found: the laptop has **hybrid graphics**:
- `card1` = NVIDIA (discrete GPU) — no physical display connectors, rendering only
- `card2` = AMD (integrated GPU) — actually drives the screen

KWin was grabbing the NVIDIA card first at boot, which cannot output to any display,
causing the black screen even after the video group was fixed.

## Changes Made

### 1. Added user to `video` group (permanent, still in effect)

```bash
sudo usermod -aG video <username>
```

Required a reboot to take effect. Verified after reboot:

```bash
groups <username>
# <username> : <username> wheel video users <groups>
```

### 2. KWin GPU override — attempted and reverted

Created `~/.config/environment.d/kwin-drm.conf` to force KWin to use the AMD GPU:

```
KWIN_DRM_DEVICES=/dev/dri/by-path/pci-<pci-address>-card
```

This broke the login entirely (no display at all), so it was deleted. File does not
currently exist.

## Current State

- `<username>` is in the `video` group ✓
- `~/.config/environment.d/kwin-drm.conf` — deleted, does not exist
- **Black screen on boot still occurs** — workaround: close and reopen laptop lid
- Root cause is KWin picking up the NVIDIA card over AMD on boot

## How To Undo the Video Group Change

```bash
sudo gpasswd -d <username> video
```

Takes effect after next login. Not recommended — the video group membership is
correct and necessary.

## Future Fix

The black screen needs a proper KWin/Wayland GPU selection fix. The `kwin-drm.conf`
approach is correct in theory but the exact device path needs to be verified first:

```bash
ls -la /dev/dri/by-path/
```

Find the entry ending in `-card` that corresponds to the AMD GPU (vendor 0x1002,
PCI address <pci-address>), then set:

```
KWIN_DRM_DEVICES=/dev/dri/by-path/pci-<pci-address>-card
```

Test carefully — a wrong path will leave no display output on boot.
