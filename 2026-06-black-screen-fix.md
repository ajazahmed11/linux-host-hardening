# System Change Log - Video Group & Black Screen Fix

**Date:** 2026-05 (initial attempts) → 2026-06-27 (resolved)
**Changed by:** Claude-assisted, approved by user
**Outcome:** Fully resolved — kernel parameter `nvidia-drm.modeset=1` fixed the black screen permanently

## Problem

Black screen after login on every boot. The workaround was closing and reopening the
laptop lid to trigger a display refresh. This was traced to KWin (the KDE window
manager) failing to access GPU hardware.

## Root Cause Investigation

Logs showed KWin failing with:

```
kwin_wayland: Failed to open drm node: /dev/dri/card1 — Permission denied
```

`/dev/dri/card1` is owned by the `video` group. The user `drelhune` was not in the
`video` group, so KWin silently failed to open the display device.

A second deeper cause was also found: the laptop has **hybrid graphics**:
- `card1` = NVIDIA (discrete GPU) — no physical display connectors, rendering only
- `card2` = AMD (integrated GPU) — actually drives the screen

KWin was grabbing the NVIDIA card first at boot, which cannot output to any display,
causing the black screen even after the video group was fixed.

## Changes Made

### 1. Added user to `video` group (permanent, still in effect)

```bash
sudo usermod -aG video drelhune
```

Required a reboot to take effect. Verified after reboot:

```bash
groups drelhune
# drelhune : drelhune wheel video users falcond libvirt docker
```

### 2. KWin GPU override — attempted and reverted

Created `~/.config/environment.d/kwin-drm.conf` to force KWin to use the AMD GPU:

```
KWIN_DRM_DEVICES=/dev/dri/by-path/pci-0000:05:00.0-card
```

This broke the login entirely (no display at all), so it was deleted. File does not
currently exist.

## Current State

- `drelhune` is in the `video` group ✓
- `~/.config/environment.d/kwin-drm.conf` — deleted, does not exist
- **Black screen on boot resolved** ✓
- `nvidia-drm.modeset=1` kernel parameter active and permanent

## Actual Fix (2026-06-27)

After further investigation, the root cause was the NVIDIA driver loading at boot
and trying to own display output without checking whether it has a physical path to
the screen. On this muxless hybrid graphics laptop, the internal display is wired
only to AMD — NVIDIA has no physical display connectors.

Enabling DRM modesetting for the NVIDIA driver hands display ownership to the
kernel's DRM layer instead of the driver. DRM correctly identifies that AMD has
the display path and gives it control. NVIDIA loads silently in the background
available for compute/rendering only.

### Fix applied

```bash
sudo grubby --update-kernel=ALL --args="nvidia-drm.modeset=1"
```

Followed by a reboot. No display issues since.

### Verify the parameter is active

```bash
cat /proc/cmdline
```

`nvidia-drm.modeset=1` should appear in the output.

## How To Undo the Video Group Change

```bash
sudo gpasswd -d drelhune video
```

Takes effect after next login. Not recommended — the video group membership is
correct and necessary.

## How To Undo the Kernel Parameter

```bash
sudo grubby --update-kernel=ALL --remove-args="nvidia-drm.modeset=1"
```

Not recommended — this will bring the black screen back.
