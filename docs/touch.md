# Automotive Touch Controller Integration

Projected Capacitive (PCAP) touch sensors are standard on automotive Center Information Displays (CID) and passenger panels. Automotive touch controllers must operate through thick curved cover glass (3–6 mm), withstand wet conditions, support gloved operation, and reject severe vehicle electromagnetic noise.

---

## 1. Physical Architecture & SerDes Routing

In remote automotive displays, the touch controller sits on the display PCB and communicates with the central SoC via the SerDes backchannel:

```mermaid
flowchart LR
    subgraph CockpitECU["Central Cockpit ECU"]
        SoC["SoC Host Processor\n(Linux / Android / QNX)"]
        SER["Serializer IC"]
        SoC <-->|I2C Master (Virtual Alias Addr)| SER
        SoC <--|Hardware GPIO IRQ| SER
    end

    subgraph SerDesLink["Automotive Cable"]
        SER <===>|Coaxial Backchannel Packets| DES
    end

    subgraph RemoteDisplay["Remote Display Assembly"]
        DES["Deserializer IC"]
        TOUCH_IC["Touch Controller IC\n(PCAP Engine)"]
        GLASS["Sensor Matrix & Cover Glass\n(Optically Bonded)"]
        
        DES <-->|Remote I2C Bus (Physical Addr)| TOUCH_IC
        TOUCH_IC -->|Hardware INT# Pin| DES
        TOUCH_IC <-->|Drive / Sense Traces| GLASS
    end
```

---

## 2. Linux Kernel Input Subsystem & Multi-Touch (MT)

Touch devices register under the Linux `input` subsystem and transmit events using **Multi-Touch (MT) Protocol Type B** (stateful slot-based tracking):

```text
Event Sequence for a Single Touch Event:
SYN_REPORT       ------ Frame Boundary ------
EV_ABS           ABS_MT_SLOT         0           (Slot 0 active)
EV_ABS           ABS_MT_TRACKING_ID  142         (Unique touch ID)
EV_ABS           ABS_MT_POSITION_X   1240        (X Coordinate)
EV_ABS           ABS_MT_POSITION_Y   580         (Y Coordinate)
EV_ABS           ABS_MT_PRESSURE     45          (Contact pressure)
SYN_REPORT       ------ End of Frame --------
```

---

## 3. Coordinate Calibration & Orientation Transformation

Displays and touch sensors may have different coordinate origins or scan directions. Alignment is handled via Device Tree or userspace calibration.

### Device Tree Coordinate Properties
Standard Linux touchscreen DT properties allow inverting or swapping axes directly in kernel space:

```dts
touchscreen@38 {
    compatible = "vendor,automotive-touch";
    reg = <0x38>;
    interrupt-parent = <&gpio1>;
    interrupts = <18 IRQ_TYPE_LEVEL_LOW>;
    reset-gpios = <&gpio1 19 GPIO_ACTIVE_LOW>;

    touchscreen-size-x = <1920>;
    touchscreen-size-y = <720>;
    touchscreen-inverted-x;       /* Inverts X axis */
    /* touchscreen-inverted-y; */
    /* touchscreen-swapped-x-y;  Swaps X and Y for portrait displays */
};
```

### Userspace Calibration Matrix (`libinput` / Wayland)
For Wayland/Weston, the coordinate transformation matrix is defined as a $3 \times 3$ affine transform:

$$\begin{bmatrix} X' \\ Y' \\ 1 \end{bmatrix} = \begin{bmatrix} c_0 & c_1 & c_2 \\ c_3 & c_4 & c_5 \\ 0 & 0 & 1 \end{bmatrix} \begin{bmatrix} X \\ Y \\ 1 \end{bmatrix}$$

* **Normal (Identity)**: `1 0 0 0 1 0`
* **Rotate 90° Clockwise**: `0 1 0 -1 0 1`
* **Rotate 180° (Invert X & Y)**: `-1 0 1 0 -1 1`

### Android Input Device Configuration (`.idc`)
On Android, create `/vendor/usr/idc/<device_name>.idc`:
```properties
touch.deviceType = touchScreen
touch.orientationAware = 1
touch.gestureMode = spots
```

---

## 4. Diagnostic Commands & Testing

```bash
# 1. Verify touch interrupt count increases when touching the screen
cat /proc/interrupts | grep -i -E "touch|i2c"

# 2. List input event nodes to identify touch handler
cat /proc/bus/input/devices

# 3. Real-time event monitoring with evtest (Linux)
evtest /dev/input/event1

# 4. Real-time event monitoring on Android
getevent -l /dev/input/event1

# 5. Monitor Wayland/Weston processed touch coordinates
libinput debug-events --show-keycodes
```

---

## 5. Touch Bring-Up Troubleshooting Matrix

| Symptom | Root Cause | Remediation |
| :--- | :--- | :--- |
| **No response; no interrupts recorded** | Touch controller held in reset, missing power rail, or wrong IRQ trigger type | Check reset GPIO state; verify `IRQ_TYPE_LEVEL_LOW` vs `IRQ_TYPE_EDGE_FALLING` in DTS. |
| **Finger moves left $\rightarrow$ cursor moves right** | X-axis inversion | Add `touchscreen-inverted-x;` to the device tree node. |
| **Finger moves horizontally $\rightarrow$ cursor moves vertically** | X/Y axes swapped (portrait vs landscape orientation mismatch) | Add `touchscreen-swapped-x-y;` to the device tree node. |
| **"Ghost" touches or cursor jumping wildly** | Electrical noise coupled from display backlight inverter or ground loop | Enable touch IC noise frequency hopping; verify optical bonding ground shield connection. |
| **Touch stops working when screen gets wet** | Water droplets creating capacitive bridges | Tune touch IC firmware water-rejection baseline algorithm. |
| **System sluggish / CPU core pegged at 100%** | Level-triggered interrupt not cleared by driver (interrupt storm) | Ensure driver reads all FIFO event bytes from touch IC in the ISR before exiting. |
