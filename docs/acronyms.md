# Automotive Display Acronyms & Terminology Glossary

A quick-reference glossary of acronyms, hardware buses, software subsystems, and automotive display industry terms.

---

## 1. Display Interfaces & Physical Standards

| Acronym | Definition | Description |
| :--- | :--- | :--- |
| **DSI** | Display Serial Interface | MIPI standard high-speed serial interface using differential clock and data lanes (D-PHY or C-PHY). |
| **CSI** | Camera Serial Interface | MIPI standard for video input (cameras), often sharing PHY architecture with DSI. |
| **LVDS** | Low-Voltage Differential Signaling | Generic physical layer differential signaling standard (ANSI/TIA/EIA-644). |
| **OpenLDI** | Open LVDS Display Interface | De-facto standard for transmitting digital video over LVDS lines (single or dual link). |
| **eDP** | Embedded DisplayPort | VESA standard internal display interface based on DisplayPort architecture with AUX channel. |
| **DP** | DisplayPort | Digital display interface standard developed by VESA for external displays. |
| **HDMI** | High-Definition Multimedia Interface | Proprietary audio/video interface standard using TMDS or FRL. |
| **TTL / RGB** | Transistor-Transistor Logic Parallel RGB | Legacy parallel video bus with separate R, G, B data lines, HSYNC, VSYNC, DE, and PCLK. |
| **A-PHY** | MIPI A-PHY | Automotive physical layer specification for long-reach high-speed SerDes (up to 15m). |
| **ASA** | Automotive SerDes Alliance | Open industry standard for automotive high-speed SerDes communication. |

---

## 2. Video Timings & Blanking Anatomy

| Acronym | Definition | Description |
| :--- | :--- | :--- |
| **PCLK** | Pixel Clock | Rate at which pixels are transferred across the interface, measured in Hz (typically MHz). |
| **HACT** | Horizontal Active | Number of visible horizontal pixels per scan line (e.g., 1920). |
| **HFP** | Horizontal Front Porch | Blanking pixels between the end of active video and the start of HSYNC. |
| **HSA / HSW** | Horizontal Sync Width | Duration of horizontal synchronization pulse in pixel clocks. |
| **HBP** | Horizontal Back Porch | Blanking pixels between the trailing edge of HSYNC and active video. |
| **HTOT** | Horizontal Total | Total pixel cycles per horizontal line: $HTOT = HACT + HFP + HSA + HBP$. |
| **VACT** | Vertical Active | Number of active visible lines per frame (e.g., 720, 1080). |
| **VFP** | Vertical Front Porch | Blanking lines between the end of active vertical frame and VSYNC. |
| **VSA / VSW** | Vertical Sync Width | Duration of vertical synchronization pulse in scan lines. |
| **VBP** | Vertical Back Porch | Blanking lines between the end of VSYNC and the start of visible lines. |
| **VTOT** | Vertical Total | Total scan lines per frame: $VTOT = VACT + VFP + VSA + VBP$. |
| **DE** | Data Enable | Signal indicating valid pixel data is present on the bus (high during active video). |
| **BPP** | Bits Per Pixel | Color depth per pixel (e.g., 16-bit RGB565, 24-bit RGB888, 30-bit RGB101010). |
| **FPS** | Frames Per Second | Vertical refresh rate of the display panel (standard automotive: 60 Hz). |

---

## 3. Graphics Subsystems & Linux DRM/KMS

| Acronym | Definition | Description |
| :--- | :--- | :--- |
| **DRM** | Direct Rendering Manager | Linux kernel subsystem responsible for interfacing with the GPU and display controllers. |
| **KMS** | Kernel Mode Setting | DRM component handling display resolution, timings, color depth, and display hardware configuration. |
| **CRTC** | Cathode Ray Tube Controller | Abstract KMS object representing a scanout engine that reads framebuffers and generates timing. |
| **Plane** | DRM Plane | Hardware layer/overlay that feeds pixel buffers into a CRTC (Primary, Overlay, Cursor). |
| **Encoder** | DRM Encoder | Hardware block converting pixel data from CRTC into an interface protocol (e.g., DSI, HDMI, DP). |
| **Connector** | DRM Connector | Abstract KMS object representing the physical output/cable to a display panel. |
| **Bridge** | DRM Bridge (`drm_bridge`) | Reusable kernel driver for an off-SoC or intermediate conversion IC (e.g., DSI-to-LVDS bridge, SerDes). |
| **Panel** | DRM Panel (`drm_panel`) | Kernel driver encapsulating panel power-up, reset sequencing, and display timings. |
| **GEM** | Graphics Execution Manager | Linux kernel memory management subsystem for allocating and managing graphics buffers. |
| **DMA-BUF** | Direct Memory Access Buffer | Linux cross-driver buffer sharing mechanism for zero-copy buffer handoff between GPU and display. |
| **VBLANK** | Vertical Blanking Interrupt | Hardware interrupt triggered when the scanout engine reaches the vertical blanking interval. |

---

## 4. SerDes & Transmission Link

| Acronym | Definition | Description |
| :--- | :--- | :--- |
| **SerDes** | Serializer / Deserializer | Pair of ICs converting parallel/wide high-speed data into high-speed serial streams and back. |
| **SER** | Serializer | Transmitter IC situated close to the SoC, converting video, audio, and control data into serial line. |
| **DES** | Deserializer | Receiver IC situated inside or near the display module, recreating the original video stream. |
| **PoC** | Power over Coax | Technique supplying DC power to remote display modules over the same coaxial signal wire. |
| **FPD-Link** | Flat Panel Display Link | Proprietary automotive SerDes technology developed by Texas Instruments (FPD-Link II/III/IV). |
| **GMSL** | Gigabit Multimedia Serial Link | Proprietary automotive SerDes technology developed by Analog Devices / Maxim (GMSL 1/2/3). |
| **APIX** | Automotive Pixel Link | Proprietary automotive SerDes technology developed by Inova Semiconductors (APIX 2/3). |
| **Backchannel** | Reverse Control Channel | Low-speed bidirectional channel multiplexed over the SerDes cable for I2C, SPI, GPIO, and interrupts. |
| **Forward Channel** | Downstream Video Channel | High-bandwidth unidirectional data stream carrying video, audio, and high-speed data from SoC to display. |
| **LOCK** | SerDes Lock State | Hardware status signal confirming the deserializer PLL has synchronized with the serializer bit clock. |
| **STP** | Shielded Twisted Pair | Two insulated copper conductors twisted together inside shielding, alternative to coaxial cable. |

---

## 5. Hardware, Panel, Backlight & Touch

| Acronym | Definition | Description |
| :--- | :--- | :--- |
| **TCON** | Timing Controller | Panel-side ASIC that converts incoming video streams (LVDS/eDP/MIPI) into row and column driver signals. |
| **PCAP** | Projected Capacitive Touch | Touch screen technology measuring capacitance changes caused by user fingers. |
| **VDD / VCC** | Logic Power Rail | Digital core power supply (typically 1.8V or 3.3V). |
| **AVDD** | Analog Power Rail | High-voltage supply for panel analog circuits and column drivers (typically 8V to 15V). |
| **VGH / VGL** | Gate High / Gate Low | Supply voltages for TFT gate drivers (VGH: +15V to +30V to turn ON, VGL: -5V to -15V to turn OFF). |
| **VCOM** | Common Voltage | Reference voltage applied to the common electrode across liquid crystal cells. |
| **PWM** | Pulse Width Modulation | Digital signal where duty cycle modulates backlight brightness. |
| **ALS** | Ambient Light Sensor | Sensor measuring vehicle cabin illumination to adjust display backlight automatically. |
| **DCS** | Display Command Set | MIPI standard command set (e.g., `set_display_on`, `sleep_out`) used to control display registers. |
| **BTA** | Bus Turnaround | MIPI DSI physical protocol operation handing bus control from host to device to read registers. |

---

## 6. Multi-OS & Windowing Concepts

| Acronym | Definition | Description |
| :--- | :--- | :--- |
| **HWC** | Hardware Composer | Android HAL module determining which layers are composited by display hardware vs GPU. |
| **SurfaceFlinger** | Android Compositor | System service accepting buffers from applications and compositing them for display. |
| **Weston** | Wayland Reference Compositor | Default compositor used in many embedded Linux and automotive Linux distributions (AGL, GENIVI). |
| **Screen** | QNX Screen Graphics Subsystem | High-performance compositing window manager and graphics framework in QNX Neutrino RTOS. |
| **GBM** | Generic Buffer Management | Mesa API providing memory allocation for EGL/DRM on Linux without X11. |
| **EGL** | Embedded Graphics Library | Khronos interface between rendering APIs (OpenGL ES, Vulkan) and native underlying window systems. |

---

## 7. Automotive Architecture & Functional Safety (FuSa)

| Acronym | Definition | Description |
| :--- | :--- | :--- |
| **ECU** | Electronic Control Unit | Embedded automotive computer controlling vehicle subsystems. |
| **CDC / Cockpit** | Cockpit Domain Controller | High-compute multi-core SoC driving multiple in-cabin displays and safety workloads. |
| **IC / Cluster** | Instrument Cluster | Safety-critical driver display presenting speedometer, warnings, telltales, and ADAS alerts. |
| **CID** | Center Information Display | Center console infotainment and HVAC control screen. |
| **HUD** | Head-Up Display | Optical projection display projecting critical driving telemetry onto the windshield. |
| **CMS** | Camera Monitor System | Digital side/rear-view mirror system replacing physical mirrors with cameras and low-latency displays. |
| **ASIL** | Automotive Safety Integrity Level | Risk classification defined by ISO 26262 (ASIL A, B, C, D; with D being the highest safety level). |
| **BIST** | Built-In Self-Test | Hardware or firmware self-test routines verifying SerDes link integrity or panel memory on boot. |
| **CRC** | Cyclic Redundancy Check | Hardware pixel or frame CRC checking engine used to detect frozen or corrupted cluster display frames. |
| **Tell-tale** | Warning Indicator Light | ISO standardized warning symbol on instrument clusters (e.g., check engine, ABS, airbag). |
