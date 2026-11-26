# SBC Controller v0 (`SBCC01-RP5`)

Power management (5V/6A), supercapacitor backup, RS-485 communication, and 9-axis IMU expansion HAT for **Raspberry Pi 4 / 5**.

> [!NOTE]
> This documentation is currently an AI-generated quick overview and may contain inaccuracies. Please verify all pinouts, jumper settings, and circuit details directly against the schematic (`SBC_controller_sch.pdf`) before assembling or powering the board.

<p align="center">
  <img width="4032" height="3024" alt="IMG_1498" src="https://github.com/user-attachments/assets/4ae0e57d-78db-4820-81dc-e5336499d53b" />
</p>


## Overview

`SBC_controller_v0` (`SBCC01-RP5`) is a 4-layer hardware HAT designed for single-board computers (**Raspberry Pi 4 Model B** and **Raspberry Pi 5**). It steps down an external `+12V` power input to `+5V` (up to `6A`) using a high-current DC-DC module, provides backup power control with a `1.5F` supercapacitor, monitors both input current and `5V` rail voltage, and integrates an RS-485 transceiver, 9-axis IMU modules, and an onboard **Seeeduino XIAO / XIAO RP2040** co-processor.

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
2. Upload **`gerber/gerber.zip`** directly (do not unzip it).
3. Confirm the auto-detected PCB parameters:
   - **Base Material**: FR-4
   - **Layers**: 4
   - **PCB Thickness**: 1.6 mm
4. Select your preferred PCB color / surface finish (e.g., Green / HASL or ENIG) and proceed to checkout.

<img width="1037" height="999" alt="image" src="https://github.com/user-attachments/assets/f7617f10-22a3-4e58-ab9e-1b2cd5ee24ca" />


## Firmware & BOM (Coming Soon)

The following files are currently being prepared and will be added in a future update:

- **Sample Firmware for Seeeduino XIAO**: Example code for monitoring `Vsense` (`5V` rail voltage) and `Isense` (`12V` input current), managing power sequences, and communicating with the Raspberry Pi.
- **PCBA BOM (Bill of Materials)**: Complete component list with part numbers, footprints, and JLCPCB / supplier references for assembly.
