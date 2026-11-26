# SBC Controller


Power management, supercapacitor backup, RS-485 communication, and 9-axis IMU expansion HAT for **Raspberry Pi 4 / 5**.

<p align="center">
  <img width="4032" height="3024" alt="IMG_1498" src="https://github.com/user-attachments/assets/4ae0e57d-78db-4820-81dc-e5336499d53b" />
</p>

## Overview

`SBC_controller_v2` is a 4-layer hardware HAT designed for single-board computers (**Raspberry Pi 4 Model B** and **Raspberry Pi 5**). It steps down an external 12V power input to 5V/3.5A, provides soft-start and brownout/backup power control using a 1.5F supercapacitor, monitors high-side input current, and integrates an RS-485 transceiver, 9-axis IMU modules, and an onboard **Seeeduino XIAO / XIAO RP2040** co-processor.

## Features

- **Raspberry Pi 4 / 5 Compatibility**: Standard 40-pin GPIO header (`J1`) compatible with Raspberry Pi 4 and Raspberry Pi 5.
- **High-Current 5V Power Supply**: Onboard 5V / 3.5A DC-DC converter module (`U5`, Kondo Kagaku `DCDC5V3.5A`) with reverse-current protection Schottky barrier diodes (`D1`, `D2` / `SX34F`).
- **Supercapacitor Backup & Power Supervisor**:
  - 1.5F / 5.4V supercapacitor (`C2`, `VEC5R4155QG`) for transient power loss buffering and safe shutdown support.
  - Voltage detector / reset IC (`U3`, `M51957B`) driving a dual P/N-channel MOSFET load switch (`Q1`, `TPC8407`).
- **High-Side Current Sensing**: `ZXCT1008FTA` current sense amplifier (`U7`) with a `20mΩ` shunt resistor (`R14`) on the `+12V` input rail.
- **Onboard Co-Processor**: Footprint for **Seeeduino XIAO / XIAO RP2040** (`U1`) for power sequence management, ADC monitoring, and SPI/UART/I2C bridging.
- **RS-485 Bus Interface**: `MAX485EESA+T` transceiver (`U2`) with `120Ω` termination (`R3`) and dual daisy-chainable 4-pin JST-XH connectors (`J2`, `J3`).
- **9-Axis IMU Support**: Supports **AE-BMX055** (`U4`) and **AE-BNO055** (`U6`) modules over I2C (`3.3V` logic).

## Schematic

The full schematic is available in [`SBC_controller_sch.pdf`](SBC_controller_sch.pdf).

<p align="center">
  <img src="docs/schematic.png" alt="SBC Controller v2 Schematic" width="800">
</p>

## Hardware Specifications & Pinout

### PCB Specifications

| Parameter | Specification |
| :--- | :--- |
| **Board Revision** | v2 (`SBCC01-RP4` / `SBCC01-RP5`) |
| **Layer Stackup** | 4-Layer (`F.Cu` / `In1.Cu` / `In2.Cu` / `B.Cu`) |
| **Board Thickness** | 1.6 mm (FR-4) |
| **Main Power Input** | +12V DC (via `J2` / `J3`) |
| **Regulated Output** | +5V DC (3.5A max), +3.3V DC |

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
| **Pin 2, 4** | `+5V` Power Rail | Power output from HAT (`D1` cathode / `JP1`) |
| **Pin 3 (`GPIO2`)** | `I2C1_SDA` | `U4` (BMX055), `U6` (BNO055), `U1` (`D4` via `JP2`) |
| **Pin 5 (`GPIO3`)** | `I2C1_SCL` | `U4` (BMX055), `U6` (BNO055), `U1` (`D5` via `JP3`) |
| **Pin 8 (`GPIO14`)** | `UART_TXD` | `U2` (`DI` via `JP5`), `U1` (`D6` via `JP5`) |
| **Pin 10 (`GPIO15`)** | `UART_RXD` | `U2` (`RO` via `JP4`), `U1` (`D7` via `JP4`) |
| **Pin 19 (`GPIO10`)** | `SPI0_MOSI` | `U1` (`D10` / `MOSI`) |
| **Pin 21 (`GPIO9`)** | `SPI0_MISO` | `U1` (`D9` / `MISO`) |
| **Pin 23 (`GPIO11`)** | `SPI0_SCLK` | `U1` (`D8` / `SCK`) |
| **Pin 24 (`GPIO8`)** | `SPI0_CE0` | `U1` (`D2` / `CS`) |
| **Pin 36 (`GPIO16`)** | `RS485_DE/RE` | `U2` (`DE`/`~RE` via `JP6`), `U1` (`D3` via `JP6`) |

### Solder Jumpers (`JP1` – `JP6`)

| Jumper | Default | Function |
| :---: | :---: | :--- |
| **`JP1`** | Open | Bypasses the MOSFET load switch (`Q1`) between the 5V rail and Raspberry Pi 5V pins |
| **`JP2`** | Open | Connects `I2C_SDA` (`GPIO2`) to XIAO `D4` |
| **`JP3`** | Open | Connects `I2C_SCL` (`GPIO3`) to XIAO `D5` |
| **`JP4`** | Open | Connects `UART_RXD` (`GPIO15`) / `MAX485 RO` to XIAO `D7` |
| **`JP5`** | Open | Connects `UART_TXD` (`GPIO14`) / `MAX485 DI` to XIAO `D6` |
| **`JP6`** | Open | Connects `RS485 DE/~RE` (`GPIO16`) to XIAO `D3` |

## Directory Structure

```text
SBC_controller_v2/
├── SBC_controller.kicad_pro       # KiCad project file
├── SBC_controller.kicad_sch       # KiCad schematic file
├── SBC_controller.kicad_pcb       # KiCad 4-layer PCB layout file
├── SBC_controller.step            # 3D STEP model of the assembled PCB
├── SBC_controller_sch.pdf         # Exported schematic (PDF)
├── gerber/                        # Manufacturing Gerber & drill files
│   └── gerber.zip                 # Ready-to-order Gerber archive
├── DesignReview/                  # Design review documents (PDF)
├── READ_ME.txt                    # Revision notes
└── <Component_Libraries>/         # Custom KiCad symbols & footprints
