---
  # Linux Host Security Hardening

  A collection of documented security changes made to a Fedora-based (Nobara) Linux daily-driver system. Each log
  covers the problem, root cause, changes made, how to verify, and how to undo.

  ## Logs

| File                                                         | What it covers                                               |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [Firewall — Spotify Port Restriction](2026-05-firewall-spotify-rules.md) | Restricting open ports to localhost using firewalld rich rules |
| [Video Group & Black Screen Investigation](2026-05-video-group-black-screen.md) | Tracing a post-login black screen to a DRM permission and hybrid GPU issue |
| [SELinux Enable Attempt](2026-05-selinux-attempt.md)         | Attempting to enable SELinux on Nobara and why it failed     |
| [Disable CPU Boost at Startup](2026-05-11-disable-cpu-boost.md) | Systemd service to disable AMD CPU boost on every boot       |
| [Lid Suspend Fix](2026-05-05-lid-suspend-fix.md)             | Fixing lid-close suspend via logind and polkit configuration |
| [dnf-automatic Security Updates](2026-05-dnf-automatic.md)   | Enabling and disabling automatic security updates            |
| **Black Screen Post login Fix**                              | Black screen on boot — root cause and fix via `nvidia-drm.modeset=1` kernel paramete |

  ## Environment

  - OS: Nobara Linux 43 (Fedora-based, KDE Plasma)
  - Hardware: ASUS ROG laptop, AMD Ryzen + NVIDIA/AMD hybrid GPU

---
