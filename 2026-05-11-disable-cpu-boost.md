# System Change Log - Disable CPU Boost at Startup

**Date:** 2026-05-11
**Changed by:** Claude-assisted, approved by user

## Problem

CPU boost (AMD Turbo) was enabled by default on the Ryzen 7 4800HS. The user wanted
boost disabled automatically at every startup without having to do it manually each time.

## Why This Was Done

CPU boost increases performance but raises heat and power consumption. Disabling it
keeps the laptop cooler and more power-efficient during normal use.

## Files Created

### `/etc/systemd/system/disable-cpu-boost.service`

A systemd oneshot service that runs at boot and writes `0` to the kernel's CPU boost
control file, disabling AMD boost.

```ini
[Unit]
Description=Disable CPU Boost at startup
After=sysinit.target

[Service]
Type=oneshot
ExecStart=/bin/sh -c 'echo 0 > /sys/devices/system/cpu/cpufreq/boost'
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```

### `~/disable-cpu-boost-setup.sh`

Helper script used to create the service file and enable it. Can be kept or deleted.

```bash
#!/bin/bash
cat > /etc/systemd/system/disable-cpu-boost.service << 'EOF'
[Unit]
Description=Disable CPU Boost at startup
After=sysinit.target

[Service]
Type=oneshot
ExecStart=/bin/sh -c 'echo 0 > /sys/devices/system/cpu/cpufreq/boost'
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reload
systemctl enable --now disable-cpu-boost.service
echo "Done. CPU boost disabled at startup."
```

## Commands Used

```bash
sudo bash ~/disable-cpu-boost-setup.sh
```

Which internally ran:

```bash
systemctl daemon-reload
systemctl enable --now disable-cpu-boost.service
```

Enabling created a symlink:
```
/etc/systemd/system/multi-user.target.wants/disable-cpu-boost.service
  → /etc/systemd/system/disable-cpu-boost.service
```

## How To Verify

Check service status:

```bash
systemctl status disable-cpu-boost.service
```

Check boost is off (should output `0`):

```bash
cat /sys/devices/system/cpu/cpufreq/boost
```

## How To Undo

Stop and disable the service, delete the file, and re-enable boost:

```bash
sudo systemctl disable --now disable-cpu-boost.service
sudo rm /etc/systemd/system/disable-cpu-boost.service
echo 1 | sudo tee /sys/devices/system/cpu/cpufreq/boost
```
