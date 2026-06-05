# System Change Log - Firewall Spotify Port Restriction

**Date:** 2026-05 (security audit session)
**Changed by:** Claude-assisted, approved by user

## Problem

A security audit found that Spotify was listening on two ports open to all network
interfaces (0.0.0.0), meaning any device on the same WiFi could potentially connect
to Spotify on this machine.

- **Port 57621** — Spotify local network discovery (Spotify Connect, lets other devices find your Spotify)
- **Port 56637** — Spotify local sync port

## Why This Was Done

Restricting these ports to localhost means:
- Spotify still works normally on the machine
- Other devices on the same WiFi cannot reach these ports
- Tradeoff: Spotify Connect (controlling PC playback from phone) no longer works

## Changes Made

Two rich rules added to firewalld (zone: FedoraWorkstation):

```bash
sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="127.0.0.1" port port="57621" protocol="tcp" accept'
sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="127.0.0.1" port port="56637" protocol="tcp" accept'
sudo firewall-cmd --reload
```

These rules allow those ports **only from localhost (127.0.0.1)**. Connections from
any other IP (including other devices on your WiFi) are blocked by default.

## Current Firewall State

Zone: FedoraWorkstation (active on <interface>)

```
services: dhcpv6-client mdns samba-client ssh
ports: 1025-65535/udp 1025-65535/tcp
rich rules:
    rule family="ipv4" source address="127.0.0.1" port port="56637" protocol="tcp" accept
    rule family="ipv4" source address="127.0.0.1" port port="57621" protocol="tcp" accept
```

## How To Verify

```bash
sudo firewall-cmd --list-all
```

Should show both rich rules listed above.

## How To Undo

Remove the two rich rules and reload:

```bash
sudo firewall-cmd --permanent --remove-rich-rule='rule family="ipv4" source address="127.0.0.1" port port="57621" protocol="tcp" accept'
sudo firewall-cmd --permanent --remove-rich-rule='rule family="ipv4" source address="127.0.0.1" port port="56637" protocol="tcp" accept'
sudo firewall-cmd --reload
```

This restores Spotify Connect (phone controlling PC playback) if needed.

## Notes

- These rules do **not** affect KDE Connect, ADB, Metasploit, Android tools, or any other app
- Port 902 (VMware) listens on all interfaces — this is normal VMware behaviour
- KDE Connect uses ports 1714–1764 which are unaffected by these rules
