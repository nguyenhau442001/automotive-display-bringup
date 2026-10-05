# Display Log Analysis Template

Use this template to dissect and record bootloader, kernel (`dmesg`), Android (`logcat`), or QNX (`slog2info`) graphics logs.

---

## Log Analysis Metadata

* **Log Reference**: `LOG-YYYYMMDD-<description>`
* **Date**: YYYY-MM-DD
* **Target OS / Environment**: [ Embedded Linux / Android / QNX Neutrino ]
* **Kernel / OS Version**: [ e.g., Linux 5.15.71-aarch64 ]
* **Display Interface**: [ MIPI DSI / LVDS / eDP / SerDes ]
* **Log Source**: [ `dmesg`, `debugfs`, `logcat -b crash`, `slog2info`, `modetest` ]

---

## 1. Problem Context

* **Observed Failure**:
  [e.g., Driver probe deferred indefinitely, FIFO underrun during first frame scanout, DSI clock lane failed to enter High-Speed mode.]
* **Trigger Event**:
  [e.g., Cold boot, resume from suspend (STR), switching screen resolutions.]

---

## 2. Sanitized Log Snippet

```text
[    1.240102] drm-display soc:display-controller: Probing display controller driver...
[    1.242310] drm-display soc:display-controller: Bound to dsi_host (DSI Host PHY ready)
[    1.251402] panel-generic-dsi dsi-panel: Probing generic DSI panel...
[    1.258900] panel-generic-dsi dsi-panel: Reset GPIO asserted
[    1.380120] panel-generic-dsi dsi-panel: Sending DCS Exit Sleep Mode (0x11)...
[    1.502300] panel-generic-dsi dsi-panel: Sending DCS Display On (0x29)...
[    1.560410] drm-display soc:display-controller: CRTC 34 atomic_commit starting
[    1.562100] [drm:drm_atomic_helper_commit_tail] *ERROR* [CRTC:34:crtc-0] vblank wait timed out
[    1.575000] drm-display soc:display-controller: *ERROR* FIFO Underrun detected on plane 0
```

---

## 3. Chronological Log Dissection

| Timestamp (s) | Subsystem / Driver | Event / Message | Significance & Interpretation |
| :--- | :--- | :--- | :--- |
| `1.240102` | `display-controller` | Probe initiated | SoC display core registers and clocks probed successfully. |
| `1.258900` | `panel-generic-dsi` | Reset GPIO toggled | Hardware reset sequence started. |
| `1.380120` | `panel-generic-dsi` | DCS Sleep Out (`0x11`) | Panel awakened; 120ms delay observed. |
| `1.562100` | `drm_atomic_helper` | **VBLANK wait timed out** | **First critical error**: CRTC timing generator did not generate a vertical blanking interrupt. |
| `1.575000` | `display-controller` | **FIFO Underrun** | Display DMA engine requested pixels faster than memory bus or porch timings permitted. |

---

## 4. Key Error Signatures & Root Cause Hypotheses

### Identified Signature:
`*ERROR* [CRTC:34:crtc-0] vblank wait timed out` coupled with `FIFO Underrun`.

### Root Cause Analysis:
1. **Clock Generation Failure**:
   * If the Pixel Clock ($PCLK$) PLL fails to lock or is blocked by a gating clock gate, the CRTC timing generator never advances scan lines. Consequently, hardware VBLANK interrupts never fire.
2. **DDR Bandwidth Starvation**:
   * High-resolution scanout plane priority set too low on the system interconnect (NoC/AXI), causing display FIFOs to empty before blanking intervals.

---

## 5. Register Dump & Hardware State

```text
# DSI Host Status Register Dump (Sanitized)
0x0000: 0x00000001  (Core Enable = 1)
0x0004: 0x00000000  (Status: Clock Lane HS NOT ready) <--- Anomaly!
0x0008: 0x00000005  (Error Register: SoT Sync Error on Data Lane 0)
```

---

## 6. Action Items & Verification Plan

1. [ ] Check DSI PHY PLL configuration registers in driver clock tree.
2. [ ] Increase horizontal blanking porch ($HBP / HFP$) in DTS to give FIFO more refill headroom.
3. [ ] Verify memory interconnect Quality-of-Service (QoS) display priority settings.
