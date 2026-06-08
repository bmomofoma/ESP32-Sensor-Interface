# ESP32-S3 Telemetry Hub (MRCU Rev A)

I built this board because I wanted a compact, reliable hardware platform that could pull data from five different sensors at the same time without turning the signal lines into a noisy mess. It’s a 4-layer board centered around an ESP32-S3 that acts as a central data hub, ideal for robotics or autonomous systems. 

---

## Why I Made These Design Choices

### Moving to a 4-Layer Board
A lot of simple projects stick to 2 layers, but running high-frequency data lines right next to power traces is a recipe for a noisy bus. 
* I made **Layer 2 a solid Ground plane** to give all the signals a tight, clean return path, which knocks down cross-talk immensely.
* **Layer 3 is a power plane**, which lets me cleanly route the 3.3V regulated line without running into routing bottlenecks.

### Powering the Array Safely
The board takes 5V from a USB-C port and drops it to 3.3V using an `AP7361C-33E` linear regulator. 
* The ESP32-S3 spikes in current when its Wi-Fi or Bluetooth radios kick on. This regulator can handle up to 1A, keeping the rail steady so the sensors don't glitch.
* For debugging safety, I put a ferrite bead, a Schottky diode, and a resettable polyfuse on the input to protect the expensive components downstream if I accidentally short something on the bench.

### Choosing Flat Pads (ENIG) over HASL
Two of the sensors—the `BMI088` IMU and the `MMC5633NJL` magnetometer—come in fine-pitch LGA and QFN packages, meaning the pins are tiny and sit entirely underneath the chip. 
* Standard HASL board finishes leave uneven mounds of solder on the pads, which causes these tiny chips to float or bridge during soldering. 
* I specified an **ENIG (Gold) finish** for manufacturing because it guarantees perfectly flat pads, making reflow much easier.

---

## The Hardware Setup

All five sensors communicate over a single shared $\text{I}^2\text{C}$ bus using **GPIO8 (SDA)** and **GPIO9 (SCL)**, pulled up by $4.7\text{ k}\Omega$ resistors.

| Sensor | Part Number | What It Measures |
| :--- | :--- | :--- |
| **IMU (6-Axis)** | BMI088 | Pitch, roll, yaw, and high-G acceleration |
| **Magnetometer** | MMC5633NJL | Compass heading orientation |
| **Pressure/Altimeter** | BMP280 | Altitude & atmospheric pressure |
| **Temp/Humidity** | SHT4x | Local environmental data |
| **Time-of-Flight (ToF)** | VL53L0X | Precision optical distance for proximity |

> 🛠️ **Hardware Note:** The VL53L0X distance sensor hardcodes itself to address `0x29` on boot, which can cause bus issues. To fix this, I routed its hardware shutdown pin (**XSHUT**) to **GPIO48** so the firmware can hard-reset the chip and dynamically change its address if needed.

---

## Layout Constraints & Debugging

* **Antenna Keep-Out:** There is a strict, cross-hatched keep-out zone on all layers around the ESP32’s PCB antenna. No copper, traces, or mounting screws can go near it, keeping the wireless signal strong.
* **Lab Debugging Headers:** I exposed the $\text{I}^2\text{C}$ lines on header **J2** and the main serial UART lines on header **J3**. This makes it easy to clip on a logic analyzer or hook up a USB-to-UART bridge to watch terminal logs while writing firmware.
* **Buttons & LEDs:** Included a reset button, a boot button, a green power LED, and a red status LED mapped to **GPIO3** for basic visual debugging.
