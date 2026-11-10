# NX-DBS3

> **High-Density Modular Development Platform**  
> *M.2 2260 Form Factor • Breakable Nucleo-Style Blade • ESP32-S3R8 • 2A Buck Power Tree • 9-DOF IMU • I2S Audio*

---

## Overview

The **NX-DBS3** is an ultra-dense, versatile embedded development platform engineered to bridge the gap between high-speed benchtop breadboard prototyping and space-constrained, high-vibration interactive builds (props, robotics, wearables, and modular IoT nodes).

> [!IMPORTANT]
> **NX Proprietary Docking Port (Non-PC Hardware Interconnect):**  
> While the board utilizes the standard mechanical **M.2 Key-E form factor and connector**, the pinout is an **independent, custom maker standard** (similar to the SparkFun MicroMod architecture). The M.2 connector routes raw battery power, high-current 5V LED rails, speaker outputs, and 3.3V GPIOs directly to custom carrier boards. **Do NOT plug this board into a personal computer's motherboard M.2 slot.**

Built on a compact **0.8mm 4-layer PCB in the M.2 2260 ($22 \times 60\text{ mm}$) envelope** with **100% single-sided component placement** (zero components on the bottom layer), the NX-DBS3 introduces a **dual-personality mechanical architecture**:
1. **The Modular Cartridge**: A standard **M.2 Key-E edge connector** allowing the board to slide directly into low-profile carrier boards, custom backplanes, or modular hilt sleds.
2. **The Breakable Blade**: Inspired by the STMicroelectronics Nucleo ST-Link snap-off design, users who do not need the M.2 interface can **snap off the gold-finger tab along perforated mousebites**, instantly shortening the board from $60\text{ mm}$ to **$52\text{ mm}$ (identical to standard Adafruit Feather length)** and exposing a clean row of high-power solder pads.
3. **The Breadboard Companion**: Dual rows of standard **2.54mm (0.1") castellated pins** running along the perimeter allow the board to sit cleanly on standard breadboards or be surface-mounted directly to carrier PCBs.
4. **Zero-Z Flat Bottom**: Because all ICs, passives, and connectors are strictly top-mounted, the bottom is completely smooth fiberglass and copper—allowing it to be mounted flush against aluminum, 3D prints, or battery packs using thermal tape without risk of short circuits.

<p align="center">
  <img src="docs/NX-DBS3_concept.png" alt="NX-DBS3 Mechanical & Architectural Concept" width="85%">
</p>
<p align="center">
  <em>NX-DBS3 Architectural Concept — M.2 2260 Form Factor with Breakable Key-E Tab & SMT Rear Power Tail.</em>
</p>

```text
                               22.0 mm
          +-------------[ Mid-Mount USB-C ]-------------+
         / [M2]                                    [M2] \
        | (JST-PH)                                       |
        | [=] GPIO                              GPIO [=] |
        | [=]                                        [=] |
        | [=]       2.54mm (0.1") Castellated        [=] |
        | [=]       Standard Breadboard Edges        [=] |
 60.0mm | [=]                                        [=] |
        |                                                |
        |                                                |
        | : : : : : : : : : : : : : : : : : : : : : : : :|  <-- PERFORATED SNAP LINE
        |                                                |      (Snaps to 52mm Feather length)
        +---\                                        /---+
             |===|===|===|===[  KEY E  ]===|===|===|===|
```

---

## Naming & Lineage

```text
N X - D B S 3
│     │ │ │
│     │ │ └─► ESP32-S3 (Dual-Core 240MHz, Vector DSP, Native USB-OTG)
│     │ └───► Blade (Breakable M.2 2260 to 52mm Feather snap-off design)
│     └─────► Development Platform (Direct sibling to the NX-ISD sensor series)
│             [Lore Easter Egg: Deep Space 3 / Starfleet Outpost Designation]
│
└───────────► Prototyping Series (Continuous Experimental Fleet Registry)
```

* **Lineage:** Direct architectural evolution of the [`NX-ISD`](https://github.com/Ghost3379/NX-ISD) (Intelligent Sensor Device). While the `NX-ISD` was purpose-built for tactical environmental and physiological sensing (BME680, MAX30102, display UI), the **`NX-DBS3`** strips away enclosure-bound sensors to provide a pure, agile, high-horsepower development core with generous "batteries-included" onboard peripherals.
* **The "NX" Philosophy:** Inspired by Starfleet's experimental registry designation ([Memory Alpha: NX](https://memory-alpha.fandom.com/wiki/NX)), the **NX** prefix denotes an active, pioneering prototyping platform where each build is an evolving hardware milestone.

---

## Core Hardware Architecture

### 1. Compute & High-Speed Memory
* **SoC:** Espressif **ESP32-S3R8** (Bare QFN-56 package, $7 \times 7\text{ mm}$).
  * Dual-Core Xtensa® 32-bit LX7 @ up to $240\text{ MHz}$.
  * Vector instructions for accelerated DSP, audio synthesis, and math modeling.
  * **8MB Integrated Octal PSRAM** inside the chip package (eliminates 11 high-speed board traces and external SI routing).
  * Native USB-OTG full-speed controller ($12\text{ Mbps}$).
* **Non-Volatile Storage:** External high-speed **QSPI Flash** (Winbond W25Q128JVPIQ, WSON-8 $6 \times 5\text{ mm}$, $16\text{MB}$).
* **Mass Storage:** High-speed **MicroSD card slot** (Molex 105162 push-pull) running in **hardware 4-bit SDMMC mode** via the ESP32-S3 DMA engine (up to $40\text{ MB/s}$ throughput for WAV/FLAC audio streaming and graphics).

---

### 2. High-Headroom Power Architecture & Power-Path

The NX-DBS3 implements a robust, autonomous **Power-Path management architecture** based on the proven Microchip AN1149 topology:

```text
  USB-C (VBUS) ───► [Schottky D2] ─────────┐
                                           │
  LiPo (VBAT)  ───► [P-MOSFET Q3] ─────────┴─► [ VIN ] (Automatic System Rail)
                          ▲                      │
                          │ Gate                 ├─► [TLV62569 Buck] ──► +3V3 (2.0A)
                    (Pulled by VBUS)             │
                                                 └─► [P-MOSFET Q1]   ──► VN (NeoPixels)
                                                           ▲
                                                           │ CTRL_NP (IO21)
```

1. **Automatic Power-Path Node (`VIN`):**
   * Sourced through Schottky diode `D2` (`NRVB230LSFT1G`) from USB `VBUS` ($5.0\text{V}$) or low-loss P-MOSFET `Q3` (`DMP3130L-7`) from battery `VBAT` ($3.7\text{–}4.2\text{V}$).
   * **USB Connected:** `VBUS` turns OFF `Q3` via its gate, completely isolating `VBAT` from the system load. `VIN` runs directly on USB power ($\approx 4.8\text{–}5.0\text{V}$).
   * **Battery Only:** When USB is unplugged, a pull-down resistor pulls `Q3`'s gate low, switching `Q3` on fully ($R_{DS(on)} \approx 30\text{ m}\Omega$) to supply `VIN` with virtually zero voltage drop.
2. **System Buck Regulator (`TLV62569DBV`):**
   * Synchronous step-down buck converter powered directly from `VIN`, delivering **$3.3\text{V} @ \mathbf{2.0\text{A}}$ continuous** with $>95\%$ efficiency.
   * Powers the ESP32-S3R8, Flash, IMU, RTC, Fuel Gauge, and external maker peripherals without thermal throttling.
3. **Autonomous LiPo Charger (`BQ25170DSGR`):**
   * Standalone linear charger powered from `VBUS` configured for $600\text{–}800\text{ mA}$ charging with NTC thermistor input.
   * **Decoupled Charging:** Because the system and LEDs are powered directly from `VIN`, charging current into the LiPo is never hijacked by system load. The battery charges cleanly and terminates accurately.
   * `STAT` pin directly drives an onboard charge LED with a dedicated cut-to-disable solder jumper (allowing zero-light deep sleep operation).
4. **ModelGauge™ Fuel Gauge (`MAX17048G+T10`):**
   * Precision I2C fuel gauge (`0x36`) monitoring `VBAT` directly. Reports cell voltage ($V_{CELL}$ in mV), State of Charge (SOC %), and charge/discharge rate over I2C without burning ADC pins.
   * Programmable `!ALERT` interrupt pin to wake the ESP32 on low battery.
5. **Smart NeoPixel Power-Cut (`VN`):**
   * High-side P-MOSFET `Q1` (`DMP3130L-7`) driven by NPN BJT `Q2` (`BC847W`) from GPIO **`CTRL_NP` (`IO21`)**.
   * Powered directly from **`VIN`**: receives $5\text{V}$ on USB and $VBAT$ on battery.
   * In Standby/Deep Sleep (`CTRL_NP = LOW`), power is cut to $0.0\text{V}$, eliminating the $\sim 1\text{ mA}$ per-LED vampire quiescent draw completely ($0.0\ \mu\text{A}$ standby!).

---

### 3. Integrated Motion, Audio, & Peripherals
* **9-DOF Motion Co-Processor:** CEVA / Hillcrest Labs **`BNO085`** with dedicated $32.768\text{ kHz}$ quartz crystal (`Y1`).
  * Runs Hillcrest's sensor-fusion algorithms on an internal ARM Cortex-M0+ core.
  * Streams quaternions, linear acceleration, tap detection, and stability classifiers over the shared I2C bus (`0x4A`, jumper-selectable to `0x4B`).
  * Hardware reset held high via onboard 10k pull-up `R14` (saves an MCU pin). Active-low interrupt `!INT_DOF` on **`IO5`**.
* **Digital Audio Power Amplifier:** Maxim Integrated **`MAX98357A`** I2S Class-D mono amplifier.
  * Delivers up to **$3.2\text{W}$ into $4\ \Omega$** directly from `VIN`.
  * Dedicated digital audio bus on pure GPIOs: Bit Clock `IO33`, Data `IO34`, Word Select `IO35`.
  * Speaker outputs routed to both edge solder pads and M.2 docking pins.
* **Addressable LED (NeoPixel) & Status:**
  * **Onboard RGB Status LED:** Ultra-compact **`XL-1010RGBC-WS2812B`** ($1.0 \times 1.0\text{ mm}$) powered from switched `VN`.
  * **Direct 3.3V GPIO Drive (No Level Shifter):** Driven directly from **`IO4`** (`NPD_IN`). Operates reliably across both battery and USB rails with no fragile level-shifter ICs required.
  * **Daisy-Chain Extension:** `DOUT` (`NPD_OUT`) is broken out to solder pads and M.2 pins to chain external LED strips with zero additional GPIOs.
  * **Discrete Status LED:** Dedicated blue blink LED (`LED_BUILTIN`) on **`IO13`** (1k resistor to GND).
* **Ultra-Low-Power Real-Time Clock:** Micro Crystal **`RV-3028-C7`** ($45\text{ nA}$ typ. current draw).
  * Extreme accuracy with internal factory-calibrated 32.768kHz crystal.
  * Connected to shared I2C (`0x52`) with active-low alarm interrupt `!INT_RTC` on **`IO6`** (wakes ESP32 from deep sleep).
  * Backup power rail `RTC_BCK` supports external supercapacitors or coin cells.

---

### 4. RF Agility & Software Antenna Switching
* **RF Switch:** Infineon **`BGS12WN6`** ultra-miniature ($0.7 \times 1.1\text{ mm}$) SPDT RF switch rated up to $6\text{ GHz}$ with $<0.3\text{ dB}$ insertion loss.
* **Power Source:** Powered from `VDD_SPI` (automatically powered down during deep sleep without an external FET).
* **Control:** Controlled via **`IO10`** (`RF_CTRL`):
  * **Low (Default):** On-board high-efficiency 2.4GHz ceramic chip antenna.
  * **High:** Micro-coaxial **U.FL / IPEX** connector for routing outside metal/carbon enclosures.

---

## Pin Allocation & Signal Matrix

The ESP32-S3 provides 35 usable GPIOs (excluding dedicated SPI flash and USB D+/D-). Strapping pins `IO45` ($VDD\_SPI$ voltage) and `IO46` (ROM boot messages) are left strictly **unconnected (NC)** for safe, foolproof boot behavior.

### Internal Subsystems (16 Dedicated GPIOs)

| Subsystem | Signal Name | ESP32-S3 Pin | Function |
| :--- | :--- | :---: | :--- |
| **Shared I2C Bus** | `SDA_1` | **IO8** | Shared with external header (BNO085 `0x4A`, RV-3028 `0x52`, MAX17048 `0x36`) |
| | `SCL_1` | **IO9** | Shared with external header |
| **Periphery Interrupts** | `!INT_DOF` | **IO5** | Motion wake-up interrupt from BNO085 (RTC-capable) |
| | `!INT_RTC` | **IO6** | Alarm / timer wake-up interrupt from RV-3028 (RTC-capable) |
| | `!ALERT` | **IO7** | Low-battery alert interrupt from MAX17048 (RTC-capable) |
| **4-bit SDMMC** | `SD_CMD` | **IO38** | Native Hardware Slot 1 SDMMC Host Controller |
| | `SD_CLK` | **MTCK (IO39)**| Native Hardware Slot 1 SDMMC Host Controller |
| | `SD_DAT0` | **MTDO (IO40)**| Native Hardware Slot 1 SDMMC Host Controller |
| | `SD_DAT1` | **MTDI (IO41)**| Native Hardware Slot 1 SDMMC Host Controller |
| | `SD_DAT2` | **MTMS (IO42)**| Native Hardware Slot 1 SDMMC Host Controller |
| | `SD_DAT3` | **IO2** | Native Hardware Slot 1 SDMMC Host Controller |
| **I2S Audio** | `BCLK` | **IO33** | Bit Clock to MAX98357A (Pure digital GPIO) |
| | `DIN` | **IO34** | Audio Serial Data to MAX98357A (Pure digital GPIO) |
| | `LRCLK` | **IO35** | Left/Right Word Select to MAX98357A (Pure digital GPIO) |
| **Status & Power Control**| `NPD_IN` | **IO4** | Direct 3.3V data drive to onboard XL-1010RGBC NeoPixel |
| | `CTRL_NP` | **IO21** | P-MOSFET gate to cut power to `VN` NeoPixel rail |
| | `USB_DETECT` | **IO12** | Senses presence of 5V USB connection |
| | `LED_BUILTIN`| **IO13** | Dedicated blue status LED |
| | `RF_CTRL` | **IO10** | SPDT RF switch toggle (Chip Antenna vs. U.FL) |
| | `BOOT` | **IO0** | Boot mode select button |

---

### External Breakout Headers (`J1` & `JP3` / M.2 Key-E)

By mapping internal peripherals primarily to non-header and dedicated digital pins, **every single pin on the breakout headers is 100% collision-free**:

#### Left Header `J1` (16 Pins)
* **Pin 16:** `RESET`
* **Pin 15:** `3.3V` (System Buck Output)
* **Pin 14:** `IO0` (Boot / General I/O)
* **Pin 13:** `GND`
* **Pin 12:** `IO17` (ADC2 CH6 / RTC / GPIO)
* **Pin 11:** `IO18` (ADC2 CH7 / RTC / GPIO)
* **Pin 10:** `IO11` (ADC2 CH0 / Touch 11 / GPIO)
* **Pin 9:** `IO14` (**SPI MOSI**)
* **Pin 8:** `IO15` (ADC2 CH4 / Touch 15 / GPIO)
* **Pin 7:** `IO16` (ADC2 CH5 / Touch 16 / GPIO)
* **Pin 6:** `IO36` (**SPI SCK**)
* **Pin 5:** `IO37` (**SPI MISO**)
* **Pin 4:** `GND` (Breadboard Ground)
* **Pin 3:** `RX` (`IO44` — Hardware UART0 RX)
* **Pin 2:** `TX` (`IO43` — Hardware UART0 TX)
* **Pin 1:** `VBAT` (Raw Battery Rail)

#### Right Header `JP3` (12 Pins)
* **Pin 1:** `VBAT` (Raw Battery Rail)
* **Pin 2:** `GND`
* **Pin 3:** `VBUS` (5V USB Input / Output)
* **Pin 4:** `IO1` (**ADC1 CH0** — Wi-Fi safe analog input / Touch 1)
* **Pin 5:** `IO3` (**ADC1 CH2** — Wi-Fi safe analog input / Touch 3)
* **Pin 6–10:** Uncommitted GPIOs / Expansion Pads
* **Pin 11:** `IO9` (**I2C SCL** — Standard Wire SCL)
* **Pin 12:** `IO8` (**I2C SDA** — Standard Wire SDA)

---

## Mechanical Specifications & M.2 Footprint Credits

* **Form Factor Standard:** M.2 2260 ($22.0\text{ mm} \times 60.0\text{ mm}$).
* **Snapped Dimensions:** $22.0\text{ mm} \times 52.0\text{ mm}$ (Adafruit Feather footprint match).
* **PCB Thickness:** **$0.8\text{ mm}$** (Mandated by M.2 specification; 4-layer JLC0816 stackup).
* **Header Spacing:** Dual-row breadboard-compatible $2.54\text{ mm}$ (0.1") pitch.
* **Component Clearance:** 100% single-sided top placement. Completely flat bottom layer.

### Open-Source M.2 Template Attribution
> [!NOTE]
> **M.2 Key A+E 2260 Mechanical Model & Footprint:**  
> The M.2 Key A+E mechanical boundary, card-edge footprint, and 3D modeling for the NX-DBS3 are based on the excellent open-source templates created by **RomanRom2**:  
> 🔗 [RomanRom2 / proto-PCIexpress-KiCad — M.2 A+E Key 2260 Template](https://zxgit.org/RomanRom2/proto-PCIexpress-KiCad/src/commit/c8dc75b26b060bb7fc4acad4a52cc10a46d00019/Templates/M.2%20A+E%20Key%20Templates/M.2%20A+E%20Key%202260)  
> *Massive props and gratitude to RomanRom2 for publishing precision KiCad PCIe and M.2 card templates for the maker and hardware engineering community.*

---

## License & Open Source Disclaimer

* **Hardware Design:** Licensed under the [CERN Open Hardware Licence Version 2 - Strongly Reciprocal (CERN-OHL-S)](https://ohwr.org/cernohl).
* **Firmware / BSP:** Licensed under the [GNU General Public License v3.0 (GPLv3)](https://www.gnu.org/licenses/gpl-3.0.en.html).
* **Engineering & Ownership:** Conceptualized, engineered, and maintained by [@Ghost3379](https://github.com/Ghost3379).
