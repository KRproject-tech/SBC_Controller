# SBC Controller v0 (`SBCC01-RP5`)

Power management (5V/6A), supercapacitor backup, RS-485 communication, and 9-axis IMU expansion HAT for **Raspberry Pi 4 / 5**.

> [!NOTE]
> This documentation is currently an AI-generated quick overview and may contain inaccuracies. Please verify all pinouts, jumper settings, and circuit details directly against the schematic (`SBC_controller_sch.pdf`) before assembling or powering the board.

<p align="center">
  <img width="4032" height="3024" alt="IMG_1498" src="https://github.com/user-attachments/assets/4ae0e57d-78db-4820-81dc-e5336499d53b" />
</p>


## Overview

`SBC_controller_v0` (`SBCC01-RP5`) is a 4-layer hardware HAT designed for single-board computers (**Raspberry Pi 4 Model B** and **Raspberry Pi 5**). It steps down an external `+12V` power input to `+5V` (up to `6A`) using a high-current DC-DC module, provides backup power control with a `1.5F` supercapacitor, monitors both input current and `5V` rail voltage, and integrates an RS-485 transceiver, 9-axis IMU modules, and an onboard **Seeeduino XIAO / XIAO RP2040** co-processor.

## Research & Publications

An earlier revision of this board paired with a **Raspberry Pi 4 Model B** was used as the onboard controller in the following research for DRL control:
<details><summary><b>Show Publications</b></summary>

````
@article{YAMANO2026105370,
title = {Deep reinforcement learning-based design with observation buffer of rolling motion for snake-like robots},
journal = {Robotics and Autonomous Systems},
volume = {199},
pages = {105370},
year = {2026},
issn = {0921-8890},
doi = {https://doi.org/10.1016/j.robot.2026.105370},
author = {Akio Yamano and Satomi Suzuki and Tsuyoshi Kimoto and Takashi Iwasa},
}
````
</details>
Movie: https://youtu.be/ZmoQswbGZXU?si=ONRCCkeVyBDt7wkz

## Features

- **Raspberry Pi 4 / 5 Compatibility**: Standard 40-pin GPIO header (`J1`) compatible with Raspberry Pi 4 and Raspberry Pi 5.
- **High-Current 5V Power Supply**: Onboard `5V / 6A` DC-DC converter module (`U8`, Murata `AE-MYMGK00506ERSR-5V0`) with reverse-current protection Schottky barrier diodes (`D1`, `D2` / `SX34F`): https://akizukidenshi.com/catalog/g/g115108/.
- **Supercapacitor Backup & Power Supervisor**:　https://akizukidenshi.com/catalog/g/g109300/
  - `1.5F / 5.4V` supercapacitor (`C2`, `VEC5R4155QG`) for transient power loss buffering and safe shutdown support.
  - Voltage detector / reset IC (`U3`, `M51957B`) driving a dual P/N-channel MOSFET load switch (`Q1`, `TPC8407`).
- **Voltage & High-Side Current Sensing**:
  - `ZXCT1008FTA` high-side current sense amplifier (`U7`) with a `20mΩ` shunt resistor (`R14`) on the `+12V` input rail (`Isense` → XIAO `D1`).
  - Resistive voltage divider (`R16`, `R18`) monitoring the `+5V` rail (`Vsense` → XIAO `D0`).
- **Onboard Co-Processor**: Footprint for **Seeeduino XIAO / XIAO RP2040** (`U1`) for power sequence management, ADC monitoring (`Vsense` / `Isense`), and SPI/UART/I2C bridging.
- **RS-485 Bus Interface**: `MAX485EESA+T` transceiver (`U2`) with `120Ω` termination (`R3`) and dual daisy-chainable 4-pin JST-XH connectors (`J2`, `J3`).
- **9-Axis IMU Support**: Supports **AE-BMX055** (`U4`) and **AE-BNO055** (`U6`) modules over I2C (`3.3V` logic): https://akizukidenshi.com/catalog/g/g116996/.

## Schematic

The full schematic is available in [`SBC_controller_sch.pdf`](SBC_controller_sch.pdf).

<p align="center">
 <img width="1779" height="1251" alt="image" src="https://github.com/user-attachments/assets/343f1095-9304-41f8-871b-b56d95aa1894" />
</p>

## Hardware Specifications & Pinout

### PCB Specifications

| Parameter | Specification |
| :--- | :--- |
| **Board Revision** | v0 (`SBCC01-RP5`) |
| **Layer Stackup** | 4-Layer (`F.Cu` / `In1.Cu` / `In2.Cu` / `B.Cu`) |
| **Board Thickness** | 1.6 mm (FR-4) |
| **Main Power Input** | +12V DC (via `J2` / `J3`) |
| **Regulated Output** | +5V DC (6A max via `MYMGK00506ERSR`), +3.3V DC |

### External Connector Pinout (`J2` & `J3` — JST XH 4-Pin)

`J2` and `J3` are wired in parallel to allow daisy-chaining power and RS-485 communication lines.

| Pin | Signal | Description |
| :---: | :--- | :--- |
| **1** | `GND` | Ground |
| **2** | `+12V` | Main DC Power Input (+12V) |
| **3** | `RS485_A` | RS-485 Non-Inverting Data (`A`) |
| **4** | `RS485_B` | RS-485 Inverting Data (`B`) |

### Raspberry Pi 40-Pin Header (`J1`) & Co-Processor (`U1`) Mapping

| RPi Pin (GPIO) | Net / Function | Connected Device |
| :--- | :--- | :--- |
| **Pin 2, 4** | `5Vout` Power Rail | Power output from HAT (`Q1` drain / `JP1`) |
| **Pin 3 (`GPIO2`)** | `I2C1_SDA` | `U4` (BMX055), `U6` (BNO055), `U1` (`D4` / `SDA`) |
| **Pin 5 (`GPIO3`)** | `I2C1_SCL` | `U4` (BMX055), `U6` (BNO055), `U1` (`D5` / `SCL`) |
| **Pin 8 (`GPIO14`)** | `UART_TXD` | `U2` (`DI` via `JP5`), `U1` (`D6` / `TX` via `JP5`) |
| **Pin 10 (`GPIO15`)** | `UART_RXD` | `U2` (`RO` via `JP4`), `U1` (`D7` / `RX` via `JP4`) |
| **Pin 19 (`GPIO10`)** | `SPI0_MOSI` | `U1` (`D10` / `MOSI` via `JP7`) |
| **Pin 21 (`GPIO9`)** | `SPI0_MISO` | `U1` (`D9` / `MISO` via `JP3`) |
| **Pin 23 (`GPIO11`)** | `SPI0_SCLK` | `U1` (`D8` / `SCK` via `JP8`) |
| **Pin 24 (`GPIO8`)** | `SPI0_CE0` | `U1` (`D2` / `nCE` via `JP2`) |
| **Pin 36 (`GPIO16`)** | `RS485_DE/RE` | `U2` (`DE`/`~RE` via `JP6`), `U1` (`D3` / `DE` via `JP6`) |

### Solder Jumpers (`JP1` – `JP8`)

| Jumper | Default | Function |
| :---: | :---: | :--- |
| **`JP1`** | Open | Bypasses the MOSFET load switch (`Q1`) between `+5V` and Raspberry Pi `5Vout` pins |
| **`JP2`** | Open | Connects `SPI0_CE0` (`GPIO8`) to XIAO `D2` (`nCE`) |
| **`JP3`** | Open | Connects `SPI0_MISO` (`GPIO9`) to XIAO `D9` (`MISO`) |
| **`JP4`** | Open | Connects `UART_RXD` (`GPIO15`) to XIAO `D7` (`RX`) / `MAX485 RO` |
| **`JP5`** | Open | Connects `UART_TXD` (`GPIO14`) to XIAO `D6` (`TX`) / `MAX485 DI` |
| **`JP6`** | Open | Connects `RS485 DE/~RE` (`GPIO16`) to XIAO `D3` (`DE`) |
| **`JP7`** | Open | Connects `SPI0_MOSI` (`GPIO10`) to XIAO `D10` (`MOSI`) |
| **`JP8`** | Open | Connects `SPI0_SCLK` (`GPIO11`) to XIAO `D8` (`SCK`) |

## Directory Structure

> **Note:** To respect third-party component library licenses, only fabrication files (Gerbers), exported schematics, and documentation are distributed in this repository.

```text
SBC_controller_v0/
├── README.md                      # Project documentation
├── SBC_controller_sch.pdf         # Exported schematic (PDF)
├── docs/                          # Images for documentation
│   ├── board_photo.jpg            # Photo of the assembled HAT
│   └── schematic.png              # Schematic preview image
└── gerber/                        # Manufacturing Gerber & drill files
    └── gerber.zip                 # Ready-to-order Gerber archive
```

## PCB Ordering (JLCPCB)

You can order this PCB directly from **JLCPCB** without extracting the archive:

1. Go to the [JLCPCB Order Page](https://cart.jlcpcb.com/quote) and click **Add gerber file**.
2. Upload **`jlcpcb/production_files/GERBER-SBC_controller.zip`** directly (do not unzip it).
3. Confirm the auto-detected PCB parameters:
   - **Base Material**: FR-4
   - **Layers**: 4
   - **PCB Thickness**: 1.6 mm
4. Select your preferred PCB color / surface finish (e.g., Green / HASL or ENIG) and proceed to checkout.

<img width="1037" height="999" alt="image" src="https://github.com/user-attachments/assets/f7617f10-22a3-4e58-ab9e-1b2cd5ee24ca" />

## PCB & PCBA Ordering (JLCPCB)

You can order both the bare PCB and **PCBA (PCB Assembly)** directly from **JLCPCB** using the files in this repository:

1. **Upload Gerber Files**:
   - Go to the [JLCPCB Order Page](https://cart.jlcpcb.com/quote) and click **Add gerber file**.
   - Upload **`jlcpcb/production_files/GERBER-SBC_controller.zip`** directly (do not unzip it).
   - Confirm the PCB parameters: **FR-4**, **4 Layers**, **1.6 mm Thickness**.
2. **Enable PCB Assembly (PCBA)**:
   - Toggle **PCB Assembly** ON.
   - Select **Both Sides** for *Assembly Side* (SMD components are placed on both `top` and `bottom` layers in `CPL-SBC_controller.csv`; alternatively, choose **Bottom Side** or **Top Side** if you plan to hand-solder one side).
3. **Upload BOM & CPL Files**:
   - Click **Add BOM File** and upload **`jlcpcb/production_files/BOM-SBC_controller.csv`**.
   - Click **Add CPL File** and upload **`jlcpcb/production_files/CPL-SBC_controller.csv`**.
4. **Confirm Parts & Placement**:
   - Basic SMD passives (`0805` resistors and `0603` capacitor) are pre-linked with **LCSC Part Numbers** and will be matched automatically.
   - For SMD ICs/diodes without pre-assigned LCSC numbers (`U2`, `U3`, `U7`, `Q1`, `D1`, `D2`, `R14`), either select compatible LCSC parts manually on the JLCPCB parts confirmation screen or mark them as **Do Not Place (DNP)** to hand-solder later.
   - Mark through-hole components, daughterboard modules (`U1`, `U4`, `U6`, `U8`, `C2`, `J1`–`J3`, `D3`), and solder jumpers (`JP1`–`JP8`) as **DNP**.
   - Review component rotations in the 3D placement preview and proceed to checkout.

## Bill of Materials (BOM) Overview

The assembly data (`BOM-SBC_controller.csv` and `CPL-SBC_controller.csv`) divides the board components into three categories:

### 1. Pre-Assigned LCSC SMD Passives (Auto-Matched in JLCPCB PCBA)

| Designator | Value | Footprint | LCSC Part # | Qty | Layer |
| :--- | :--- | :--- | :--- | :---: | :--- |
| `C1` | `10µF` | `0603` (1608 Metric) | `C19702` | 1 | Bottom |
| `R10` | `100Ω` | `0805` (2012 Metric) | `C17408` | 1 | Top |
| `R3` | `120Ω` | `0805` (2012 Metric) | `C17437` | 1 | Bottom |
| `R1, R16, R18` | `330Ω` | `0805` (2012 Metric) | `C17630` | 3 | Bottom (`R1`), Top (`R16, R18`) |
| `R2, R4, R5` | `680Ω` | `0805` (2012 Metric) | `C17798` | 3 | Bottom |
| `R11, R12` | `1kΩ` | `0805` (2012 Metric) | `C17513` | 2 | Top |
| `R13` | `1.2kΩ` | `0805` (2012 Metric) | `C17379` | 1 | Bottom |
| `R9` | `4.7kΩ` | `0805` (2012 Metric) | `C17673` | 1 | Bottom |
| `R6, R7, R8, R15` | `10kΩ` | `0805` (2012 Metric) | `C17414` | 4 | Bottom (`R6–R8`), Top (`R15`) |

### 2. SMD ICs, Diodes & Shunt Resistor (Manual LCSC Selection or Hand-Soldering)

| Designator | Part / Value | Footprint | Layer | Description |
| :--- | :--- | :--- | :--- | :--- |
| `R14` | `20mΩ` | `2512` (6332 Metric) | Top | High-side current sense shunt resistor |
| `D1, D2` | `SX34F` | `SODFL4725X110N` | Bottom (`D1`), Top (`D2`) | Schottky barrier diodes |
| `Q1` | `TPC8407` | `SOIC-8` | Bottom | Dual P/N-channel MOSFET load switch |
| `U2` | `MAX485E` | `SOIC-8` | Bottom | RS-485 transceiver |
| `U3` | `M51957B` | `SOIC-8` | Bottom | Voltage detector / system reset IC |
| `U7` | `ZXCT1008FTA` | `SOT23-3` / `SOIC` | Bottom | High-side current sense monitor |

### 3. Modules, Connectors & Jumpers (Hand-Soldered After PCBA)

| Designator | Part / Module | Qty | Description |
| :--- | :--- | :---: | :--- |
| `U1` | Seeeduino XIAO / XIAO RP2040 | 1 | Onboard co-processor module (SMD/pin header) |
| `U4` | AE-BMX055 | 1 | 9-axis IMU module (Akizuki Denshi) |
| `U6` | AE-BNO055 | 1 | 9-axis IMU module (Akizuki Denshi) |
| `U8` | AE-MYMGK00506ERSR-5V0 | 1 | Murata 5V / 6A DC-DC converter module |
| `C2` | `1.5F` (`VEC5R4155QG`) | 1 | 5.4V backup supercapacitor |
| `J1` | 40-Pin GPIO Header | 1 | Raspberry Pi stacking header |
| `J2, J3` | JST XH 4-Pin (`S4B-XH-A-1`) | 2 | Horizontal connectors for `+12V` & RS-485 |
| `D3` | `3mm LED` | 1 | Power indicator LED |
| `JP1`–`JP8` | Solder Jumpers (Open) | 8 | PCB pads only (no physical component required) |

## Firmware (Coming Soon)

The following files are currently being prepared and will be added in a future update:

- **Sample Firmware for Seeeduino XIAO**: Example code for monitoring `Vsense` (`5V` rail voltage) and `Isense` (`12V` input current), managing power sequences, and communicating with the Raspberry Pi.

## Known Issues & Future Improvements

- **Decoupling Capacitors (Bypass Capacitors)**: Decoupling capacitors are currently omitted near the power pins of the onboard ICs (`U2` MAX485E, `U3` M51957B, `U7` ZXCT1008FTA, etc.) and will be added in the next hardware revision to improve power rail stability and noise immunity.
