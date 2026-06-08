# MRCU Rev A: 4-Layer ESP32-S3 Telemetry & Sensor Interface Hub

An optimized, small-footprint mixed-signal data acquisition platform designed to aggregate environmental, inertial, and spatial telemetry for autonomous systems and robotics.

---

## 1. Project Specifications & Overview
The **MRCU Rev A** was designed to solve a core problem in edge-compute and robotics: routing multiple high-frequency sensors to a single processor while managing power distribution, maintaining clean signal integrity, and keeping a compact form factor.

* **Core Processor:** ESP32-S3-WROOM-1 (Dual-core Xtensa LX7, native USB-JTAG debugging)
* **PCB Stackup:** 4-Layer, FR4, 1.6mm thickness, standard 1oz copper
* **Target Domain:** Localized telemetry aggregation for autonomous hardware and robotics platforms
* **Primary Bus Architecture:** Unified, high-speed 3.3V $\text{I}^2\text{C}$ running at 400kHz

---

## 2. Engineering Challenges & Design Trade-offs

### Challenge 1: Electromagnetic Interference (EMI) & Bus Cross-Talk
Routing high-frequency $\text{I}^2\text{C}$ lines alongside raw power paths on a standard 2-layer board introduces massive parasitic noise and signal degradation.
* **The Solution:** Implemented a **4-layer stackup** with an intentional, dedicated layer assignment. Layer 2 is an uninterrupted, solid **Ground (GND) plane**, guaranteeing an incredibly tight return path for signal loops and canceling stray EMI. Layer 3 acts as a **split power plane** for clean DC distribution.

### Challenge 2: Thermal and Voltage Stability for Sensor Arrays
The board steps down an external 5V VBUS source to a clean 3.3V logic rail. Standard linear regulators can overheat or experience voltage sag when an ESP32-S3 initializes its Wi-Fi/BLE radio blocks simultaneously with sensor polling.
* **The Solution:** Chosen a high-output `AP7361C-33E` low-dropout (LDO) regulator capable of supplying up to $1\text{ A}$ continuously. The 5V input rail is hardened against noise and surges via a `BLM21PG220` ferrite bead, an active `1N5819WS` Schottky protection diode, and a resettable polyfuse (`F1`) to save downstream components during debugging.

### Challenge 3: Advanced Leadless Footprints & Design for Manufacturability (DFM)
To save board real estate, ultra-compact sensors like the `BMI088` IMU and `MMC5633NJL` magnetometer were selected. These use fine-pitch LGA and QFN packages where pins sit entirely underneath the component body.
* **The Solution:** The manufacturing package dictates an **Electroless Nickel Immersion Gold (ENIG)** surface finish. Unlike standard HASL, ENIG guarantees perfectly flat coplanar pads, which prevents component shifting or solder bridging during reflow.

---

## 3. System Architecture & Telemetry Register

The design maps all primary telemetry hardware onto a centralized $\text{I}^2\text{C}$ bus controlled by **GPIO8 (SDA)** and **GPIO9 (SCL)**, pulled high to the 3.3V rail via a dual $4.7\text{ k}\Omega$ resistor network (`R7`/`R8`).

### Integrated Sensor Suite Map

| RefDes | Sensor Type | Component | Default Address | Engineering Purpose |
| :--- | :--- | :--- | :--- | :--- |
| **U2** | Barometric/Alt | BMP280 | `0x76` / `0x77` | Altimeter tracking and ambient pressure data |
| **U3** | Time-of-Flight | VL53L0X | `0x29` *(Fixed)* | Precision optical distance and collision avoidance |
| **U6** | Environmental | SHT4x | `0x44` | High-accuracy local temperature and humidity monitoring |
| **U8** | Magnetometer | MMC5633NJL | `0x30` | Electronic compass for magnetic heading orientation |
| **U9** | 6-Axis IMU | BMI088 | `0x18` / `0x68` | Inertial navigation, pitch/roll/yaw, and high-G tracking |

> ⚠️ **Hardware Bug Mitigation:** The `VL53L0X` laser sensor defaults to a fixed boot address (`0x29`), creating initialization conflicts if secondary devices collide. To resolve this, its active-low hardware shutdown pin (**XSHUT**) was isolated and routed straight to **GPIO48**. The firmware pulls this line low during boot to force a hard reset and dynamically assign alternate bus addresses if required.

---

## 4. Physical Layout Constraints & RF Performance

### Antenna Keep-Out Zone
A major constraint of embedding wireless microcontrollers is maintaining antenna performance. A cross-hatched **KEEP-OUT ZONE** was implemented on all four layers directly underneath and surrounding the ESP32-S3's PCB antenna trace. No copper pours, structural mounting screws, or enclosure components are allowed within this region to eliminate signal attenuation or parasitic detuning.

### Testing & Debugging Infrastructure
To make physical verification fast, the layout integrates:
* **J2 (Expansion Header):** Breakout pins exposing `3V3`, `SDA`, `SCL`, and `GND` for probing with an oscilloscope or a logic analyzer.
* **J3 (Debug Serial UART):** Dedicated header mapping to UART0 (`TXD0`/`RXD0`) for terminal logging via a USB-to-UART bridge converter.
* **Hardware Interactivity:** Real-time tactile switches for `RESET` and `BOOT` alongside a software-programmable status LED (`D1`) on **GPIO3**.

---

## 5. Production & Fabrication Specifications

The following layer configuration must be preserved when exporting manufacturing Gerbers:

```text
PCB FABRICATION DATA:
----------------------------------------------------------------------
Layer 1 (Top Copper):     High-Speed Routing, RF Trace, Decoupling (1 oz)
Layer 2 (Inner Plane 1):  Dedicated Ground Return Plane (Solid GND)
Layer 3 (Inner Plane 2):  Power Routing (Split 5V and 3.3V Planes)
Layer 4 (Bottom Copper):  Auxiliary Signal Routing, Ground Fills (1 oz)

MANUFACTURING PARAMS:
----------------------------------------------------------------------
Base Material:      FR4 (Standard)
Board Thickness:    1.6mm
Solder Mask:        Matte Black
Silkscreen:         White
Surface Finish:     ENIG (Electroless Nickel Immersion Gold) - Required
Min Trace / Space:  0.15mm / 0.15mm
