# Panel Timings & Video Blanking Calculations

Correct video timing configuration is the cornerstone of display bring-up. Mismatches between SoC timing generator registers and panel timing specifications cause tearing, image shifting, FIFO underruns, or completely black screens.

---

## 1. Frame Raster Anatomy

A video frame consists of an active display region surrounded by horizontal and vertical blanking intervals:

```text
|<--------------------------------- HTOT --------------------------------->|
+--------------------------+--------+-------------+--------+---------------+
|                          |  HFP   |  HSA (HSW)  |  HBP   |               |
+--------------------------+--------+-------------+--------+---------------+
|                          |        |             |        | ^             |
|   ACTIVE DISPLAY AREA    | Blank  | Sync Pulse  | Blank  | | VACT        |
|      (HACT x VACT)       |        |             |        | v             |
|                          |        |             |        |---------------+
|                          |        |             |        | VFP (Blank)   |
|                          |        |             |        |---------------+
|                          |        |             |        | VSA (VSync)   |
|                          |        |             |        |---------------+
|                          |        |             |        | VBP (Blank)   |
+--------------------------+--------+-------------+--------+---------------+
                                                             VTOT
```

### Definitions

| Symbol | Parameter | Unit | Description |
| :--- | :--- | :--- | :--- |
| **HACT** | Horizontal Active | Pixels | Number of visible horizontal pixels across one scanline. |
| **HFP** | Horizontal Front Porch | Pixels | Blank pixels between the end of visible pixels and HSYNC start. |
| **HSA / HSW** | Horizontal Sync Width | Pixels | Width of the horizontal synchronization pulse. |
| **HBP** | Horizontal Back Porch | Pixels | Blank pixels between HSYNC trailing edge and next visible pixels. |
| **HTOT** | Horizontal Total | Pixels | Total scanline width: $HTOT = HACT + HFP + HSA + HBP$. |
| **VACT** | Vertical Active | Lines | Number of visible horizontal scanlines per frame. |
| **VFP** | Vertical Front Porch | Lines | Blank lines between the end of the active frame and VSYNC start. |
| **VSA / VSW** | Vertical Sync Width | Lines | Duration of the vertical synchronization pulse. |
| **VBP** | Vertical Back Porch | Lines | Blank lines between VSYNC trailing edge and start of active frame. |
| **VTOT** | Vertical Total | Lines | Total vertical frame lines: $VTOT = VACT + VFP + VSA + VBP$. |

---

## 2. Mathematical Timing Formulas

### 1. Total Dimensions
$$HTOT = HACT + HFP + HSA + HBP$$
$$VTOT = VACT + VFP + VSA + VBP$$

### 2. Pixel Clock (PCLK) Calculation
Given a target refresh rate $FPS$ (e.g., 60 Hz):
$$PCLK\ (\text{Hz}) = HTOT \times VTOT \times FPS$$

### 3. Vertical Refresh Rate Calculation
Given a fixed hardware oscillator or PLL generating $PCLK$:
$$FPS\ (\text{Hz}) = \frac{PCLK}{HTOT \times VTOT}$$

### 4. MIPI DSI Physical Bit Rate per Lane
For a display with $N$ data lanes and color depth $BPP$ (typically 24 for RGB888):
$$\text{Data Rate per Lane (bps)} = \frac{PCLK \times BPP}{N}$$
$$\text{D-PHY HS Byte Clock (Hz)} = \frac{\text{Data Rate per Lane}}{8}$$
$$\text{D-PHY HS Differential Clock (Hz)} = \frac{\text{Data Rate per Lane}}{2}$$

---

## 3. Data Structures: Linux DRM vs. Device Tree

### Linux Kernel `struct drm_display_mode`
In C drivers, DRM uses cumulative sync boundaries rather than standalone porch values:

```c
static const struct drm_display_mode custom_auto_panel_mode = {
    .clock        = 148500,           /* Pixel clock in kHz (148.5 MHz) */
    .hdisplay     = 1920,             /* HACT */
    .hsync_start  = 1920 + 88,        /* HACT + HFP */
    .hsync_end    = 1920 + 88 + 44,   /* HACT + HFP + HSA */
    .htotal       = 1920 + 88 + 44 + 148, /* HTOT = 2200 */
    .vdisplay     = 1080,             /* VACT */
    .vsync_start  = 1080 + 4,         /* VACT + VFP */
    .vsync_end    = 1080 + 4 + 5,     /* VACT + VFP + VSA */
    .vtotal       = 1080 + 4 + 5 + 36,    /* VTOT = 1125 */
    .flags        = DRM_MODE_FLAG_NHSYNC | DRM_MODE_FLAG_NVSYNC,
};
```

### Device Tree `display-timings` Node
In Device Tree bindings, porches and sync widths are declared independently:

```dts
display-timings {
    native-mode = <&timing0>;
    timing0: timing-1920x1080 {
        clock-frequency = <148500000>; /* 148.5 MHz */
        hactive         = <1920>;
        vactive         = <1080>;
        hfront-porch    = <88>;
        hsync-len       = <44>;
        hback-porch     = <148>;
        vfront-porch    = <4>;
        vsync-len       = <5>;
        vback-porch     = <36>;
        hsync-active    = <0>;         /* 0: Active Low, 1: Active High */
        vsync-active    = <0>;
        de-active       = <1>;
        pixelclk-active = <1>;         /* Sample on positive edge */
    };
};
```

---

## 4. Common Timing Issues & Diagnostic Table

| Symptom | Probable Cause | Corrective Action |
| :--- | :--- | :--- |
| **Image shifted to the right** | `HBP` too large or `HFP` too small | Decrease HBP, increase HFP by the same amount to keep HTOT unchanged. |
| **Image shifted to the left** | `HBP` too small | Increase HBP, decrease HFP. |
| **Image shifted down** | `VBP` too large | Decrease VBP, increase VFP. |
| **Diagonal tearing / rolling screen** | `VSYNC` frequency out of panel lock range, or sync polarity inverted | Verify $VTOT$ calculation; toggle `DRM_MODE_FLAG_PVSYNC` $\leftrightarrow$ `DRM_MODE_FLAG_NVSYNC`. |
| **Black screen with backlight ON** | PCLK frequency outside panel TCON PLL lock window | Verify panel datasheet Min/Typ/Max PCLK limits; adjust porch total. |
| **FIFO Underrun in kernel log** | Memory bandwidth insufficient or $HTOT$ blanking interval too tight | Increase $HFP$ / $HBP$ to allow display DMA FIFO refill time; raise memory interconnect priority. |
| **Flickering pixels / sparkling noise** | PCLK sampling edge inverted | Invert `pixelclk-active` (change clock edge between rising and falling edge). |
