# Sanitized Log Samples Directory

This directory stores representative, sanitized diagnostic logs from real-world display bring-up scenarios.

---

## 1. Sanitization & Anonymization Rules

To ensure strict compliance with confidentiality policies and non-disclosure agreements (NDAs), all logs added here **MUST** be sanitized before committing:

1. **NO Customer or OEM Names**: Replace vehicle manufacturer, Tier-1 supplier, or model names with generic placeholders (e.g., `OEM_X`, `VEHICLE_PLATFORM_A`).
2. **NO Internal Codewords**: Replace internal project codes, proprietary board revisions, and confidential silicon codenames with generic terms (e.g., `auto-soc-gen1`, `fpd-serializer-v2`).
3. **NO Hardware Identifiers**:
   * Sanitize MAC addresses (`XX:XX:XX:XX:XX:XX`).
   * Sanitize Serial Numbers and Chip eFuses (`SERIAL_NUMBER_REDACTED`).
   * Sanitize Vehicle Identification Numbers (VINs).
4. **NO Private File Paths**: Strip internal developer usernames, private corporate network paths, and proprietary build server directories (`/home/developer/...` $\rightarrow$ `/home/build/...`).
5. **DO Preserve Technical Indicators**: Keep timing parameters, register values, error messages, and stack traces intact as they provide the diagnostic value.

---

## 2. Recommended Log Categories & File Naming

Organize log files using descriptive naming conventions:

* `dmesg-dsi-probe-deferred.txt`: Kernel log showing probe deferral loops due to missing regulator or clock.
* `dmesg-vblank-timeout.txt`: Kernel log showing atomic commit timeout when CRTC clocks fail to start.
* `drm-debugfs-state-dump.txt`: Complete dump of `/sys/kernel/debug/dri/0/state`.
* `modetest-connector-dump.txt`: Output of `modetest -M <driver> -c` showing supported display modes and connector status.
* `qnx-slog2-display-init.txt`: Sanitized QNX Neutrino `slog2info` startup log for display drivers.
* `android-dumpsys-surfaceflinger.txt`: Android `dumpsys SurfaceFlinger` output showing plane composition and refresh timing.
* `i2c-register-dump-serializer.txt`: Output of `i2cdump` capturing SerDes link configuration registers.
