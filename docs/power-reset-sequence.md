# Power-On & Reset Sequencing

Adhering strictly to manufacturer power-on and power-off timing sequences prevents physical damage (CMOS latch-up), visual glitches (white flashes), and intermittent initialization failures in automotive display modules.

---

## 1. Power-On Sequencing

Driving high-speed differential signals or digital control lines into an unpowered panel driver IC forward-biases internal electrostatic discharge (ESD) protection diodes, leading to current leakage, latch-up, or permanent hardware failure.

```text
VDD (Logic: 3.3V / 1.8V)
      __________________________________________________________________________
_____/
     |<- t1 ->|
AVDD (Analog: ~10V - 12V)
               _________________________________________________________________
______________/
              |<- t2 ->|
VGH / VGL (Gate Drivers)
                        ________________________________________________________
_______________________/
                       |<- t3 ->|
RESET_N (Panel Reset Pin)
                                 _______________________________________________
________________________________/
                                |<- t4 ->|
Video Data Stream (DSI / LVDS / eDP)
                                          ======================================
-----------------------------------------<      Valid Video Data Stream         >
                                          ======================================
                                         |<- t5 ->|
Backlight Enable (BL_EN / PWM)
                                                   _____________________________
__________________________________________________/
```

### Power-On Timing Parameter Reference

| Parameter | Description | Typical Constraint | Rationale / Risk |
| :--- | :--- | :--- | :--- |
| **$t_1$** | $V_{DD}$ stable to $AV_{DD}$ enable | $5\,\text{ms} \le t_1 \le 50\,\text{ms}$ | Logic circuits must be initialized before high-voltage analog rail powers up. |
| **$t_2$** | $AV_{DD}$ stable to $V_{GH}/V_{GL}$ enable | $5\,\text{ms} \le t_2 \le 20\,\text{ms}$ | Prevents unconstrained high gate voltages across unpowered TFT channels. |
| **$t_3$** | Power rails stable to $RESET\_N$ deassertion | $10\,\text{ms} \le t_3 \le 50\,\text{ms}$ | Ensures internal power-on reset (POR) circuits settle completely. |
| **$t_4$** | $RESET\_N$ High to Start of Video Stream | $10\,\text{ms} \le t_4 \le 120\,\text{ms}$ | Panel internal state machine requires time to process boot ROM / DCS `0x11` Sleep Out. |
| **$t_5$** | Valid Video Stream to Backlight Enable ($BL\_EN$) | $t_5 \ge 200\,\text{ms}$ (10+ valid frames) | **Critical anti-glitch delay**: Turning on backlight before video causes a blinding white or distorted flash. |

---

## 2. Power-Off Sequencing

The power-off sequence is the **exact inverse** of the power-on sequence. Turning off power rails while the backlight is still illuminated will reveal ugly discharge artifacts and static ghost lines.

```text
Backlight Enable (BL_EN)
________
        \_______________________________________________________________________
        |<- t6 ->|
Video Data Stream
=================
                 \--------------------------------------------------------------
                 |<- t7 ->|
RESET_N
                          \_____________________________________________________
                          |<- t8 ->|
VGH / VGL & AVDD
                                    \___________________________________________
                                    |<- t9 ->|
VDD (Logic Supply)
                                              \_________________________________
```

| Parameter | Description | Typical Constraint | Purpose |
| :--- | :--- | :--- | :--- |
| **$t_6$** | Backlight Disabled to Stop of Video Stream | $t_6 \ge 150\,\text{ms}$ | Ensures LEDs are fully extinguished before pixel clocks cease. |
| **$t_7$** | Stop of Video to $RESET\_N$ assertion | $t_7 \ge 10\,\text{ms}$ | Puts panel TCON into safe shutdown state. |
| **$t_8$** | $RESET\_N$ Low to Gate/Analog rails power down | $t_8 \ge 10\,\text{ms}$ | Discharges gate capacitors safely. |
| **$t_9$** | Analog rails down to $V_{DD}$ logic power down | $t_9 \ge 10\,\text{ms}$ | Prevents parasitic SCR latch-up. |

---

## 3. Linux Kernel DRM Panel Lifecycle Callbacks

The Linux DRM framework maps these sequences cleanly into `drm_panel_funcs`:

```c
static int sample_panel_prepare(struct drm_panel *panel)
{
    /* 1. Turn on Logic and Analog Regulators */
    gpiod_set_value_cansleep(priv->reg_en_gpio, 1);
    msleep(20);

    /* 2. Deassert Reset GPIO */
    gpiod_set_value_cansleep(priv->reset_gpio, 0); /* Active Low: 0 = High */
    msleep(120); /* Wait for Sleep-Out */

    /* 3. Send DCS initialization commands */
    mipi_dsi_dcs_exit_sleep_mode(dsi);
    msleep(120);
    return 0;
}

static int sample_panel_enable(struct drm_panel *panel)
{
    /* Video stream is already running now! */
    /* 4. Send Display On command */
    mipi_dsi_dcs_set_display_on(dsi);
    msleep(50);

    /* 5. Enable Backlight (handled automatically by backlight subsystem) */
    return 0;
}
```

---

## 4. Fast-Boot / Early Display Handoff in Automotive

Automotive cockpits have strict boot-time constraints (e.g., Rear-View Camera must appear $<2.0\,\text{s}$ from ignition):

1. **Bootloader Splash Handoff**:
   * The bootloader (U-Boot, Little Kernel, or Coreboot) powers up the display rails, configures the panel, and displays a splash logo.
   * When the Linux or QNX kernel boots, the display driver must **seamlessly take over** without toggling power rails or re-asserting reset, which would cause an unacceptable black blink.
2. **Continuous Clock / Seamless Scanout**:
   * The kernel DRM driver probes the hardware, detects that the CRTC and DSI PHY are already running, and skips the hardware initialization phase, keeping the frame buffer active until userspace compositors take over.
