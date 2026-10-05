# Bring-Up Case Study Template

Use this template to document real-world display bring-up issues, investigations, root cause analyses, and solutions. Remember to keep all descriptions vendor-neutral and sanitize proprietary data.

---

## Case Overview

* **Case ID**: `CASE-YYYYMMDD-<short-description>`
* **Date**: YYYY-MM-DD
* **Author / Engineer**:
* **Status**: [ Open | Under Investigation | Resolved | Workaround ]

---

## 1. Hardware & System Setup

| Component | Specification / Part Model | Interface Protocol |
| :--- | :--- | :--- |
| **SoC / Host Platform** | [e.g., Automotive SoC Quad-Core A53] | [e.g., MIPI DSI 4-Lane] |
| **Bridge / Serializer IC** | [e.g., Automotive FPD-Link / GMSL Serializer] | [e.g., DSI to High-Speed Serial] |
| **Harness / Cable** | [e.g., 5-meter 50-ohm Coax with Mini-FAKRA] | [e.g., Forward Video + PoC + Backchannel] |
| **Remote Deserializer** | [e.g., Automotive Deserializer] | [e.g., Serial to Dual-Link OpenLDI] |
| **Display Panel** | [e.g., 12.3" TFT-LCD 1920x720 60Hz] | [e.g., Dual-channel LVDS] |
| **Operating System** | [e.g., Linux 5.15 / Android 13 / QNX 7.1] | Kernel DRM / KMS Driver |

---

## 2. Issue Description & Symptoms

### Visual Symptoms
* [Describe what appears on the screen: Black screen, flashing, color distortion, shifted image, rolling raster, etc.]

### Reproduction Rate
* [e.g., 100% on cold boot / Intermittent (1 in 5 reboots) / Temperature dependent]

### Environmental Conditions
* [e.g., Room temperature bench / Chamber at -40°C or +85°C]

---

## 3. Initial Hypotheses

1. *Hypothesis 1*: [e.g., Reset pin timing violation during power-on sequence.]
2. *Hypothesis 2*: [e.g., LVDS VESA vs JEIDA mapping mismatch in device tree.]
3. *Hypothesis 3*: [e.g., PCLK PLL frequency outside panel TCON lock range.]

---

## 4. Diagnostic & Measurement Steps

### Hardware Measurements
* **Power Rails**:
  * $V_{DD}$ (Logic): Measured `____ V` (Expected: `____ V`)
  * $AV_{DD}$ (Analog): Measured `____ V` (Expected: `____ V`)
  * $V_{GH} / V_{GL}$: Measured `____ V / ____ V`
* **Oscilloscope Waveforms**:
  * [Attach or describe signal captures: Reset deassertion vs power rail stability, PCLK frequency, differential swing]
* **SerDes Link Status**:
  * LOCK Pin State: [ High / Low / Toggling ]
  * Link Error Registers: [ Register dumps and decoded values ]

### Software & Driver Inspection
* `modetest` Output:
  ```bash
  # Command output or connector status
  ```
* Kernel / Graphics Log Snippets:
  ```text
  # Paste relevant dmesg or driver log lines
  ```

---

## 5. Root Cause Analysis

* **Explanation**:
  [Explain why the failure occurred. Connect the observed symptom with the underlying hardware, timing, or driver mismatch.]

---

## 6. Resolution & Code Changes

### Fix Description
* [Detail the fix: DTS timing modification, driver delay insertion, register adjustment, or HW rework.]

### Code Diff / Patch
```diff
--- a/arch/arm64/boot/dts/vendor/display-panel.dtsi
+++ b/arch/arm64/boot/dts/vendor/display-panel.dtsi
@@ -25,7 +25,7 @@
-       data-mapping = "vesa-24";
+       data-mapping = "jeida-24";
```

---

## 7. Verification & Lessons Learned

* **Verification Results**:
  * [Describe how the fix was validated: e.g., 100 power cycle test without failure, thermal chamber validation.]
* **Key Takeaway / Rule of Thumb**:
  * [What design guideline or bring-up checklist item does this reinforce for future projects?]
