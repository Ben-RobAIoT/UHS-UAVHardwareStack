# 🚁 ESP32-S3 UAV Gateway

> A central control and communication board for UAVs, utilizing the **ESP32-S3** microcontroller paired with the **SIM7670G 4G Cat 1** module. It acts as an intermediary gateway for transmitting and receiving long-range MAVLink data between the Ground Station and the Flight Controller.

---

## 📌 Introduction

The **ESP32-S3 UAV Gateway** project is designed to extend UAV communication capabilities beyond the limits of traditional RF signals (2.4GHz/5.8GHz) by integrating 4G LTE Cat 1 cellular connectivity. This allows the UAV to transmit and receive MAVLink packets over much greater distances compared to conventional control methods, while maintaining flexible programmability thanks to the ESP32-S3 core.

The board is designed using **EasyEDA** and optimized for mounting on carbon fiber drone frames; it minimizes signal interference by utilizing external antennas connected via U.FL/IPEX connectors.

---

## ⚙️ Key Features

- **ESP32-S3** microcontroller with integrated 4MB Flash and 2MB PSRAM.
- Supports **WiFi / BLE** connectivity via external antennas (U.FL) to reduce interference.
- Integrated **SIM7670G 4G Cat 1** module for global long-range communication.
- **USB Type-C** port supports direct code uploading without the need for an auxiliary bridge IC.
- Stable, high-efficiency **3.3V Buck converter (SY8089A)** power circuit.
- Comprehensive ESD protection, USB signal noise suppression, and a safe startup delay circuit.
- Reset and Boot buttons for manual firmware flashing.

---

## 🧩 Component Summary Table

| **No.** | **Component Name / Symbol** | **Keyword / LCSC Code (for EasyEDA search)** | **Function & Circuit Role** |
| --- | --- | --- | --- |
| **1** | **ESP32-S3-MINI-1U-N4R2** | `C22356044` (LCSC Code) | **Central Microcontroller:** Features integrated 4MB Flash and 2MB PSRAM; the U.FL connector version allows for an external antenna to avoid signal interference caused by the drone's carbon fiber frame. |
| **2** | **16-Pin USB Type-C Port** | `Type-C 16P` or `C165948` | **Connection & Programming Port:** Provides 5V power input and D+/D- data lines for direct firmware uploading to the ESP32-S3 without needing an auxiliary converter IC. |
| **3** | **Type-C CC Pin Pull-down Resistor** | `5.1k` (0603 or 0402) | **Power Detection:** Pulls the Type-C port's CC1 and CC2 pins to GND to identify the device as a power consumer (Sink), triggering the charger to supply 5V power. |
| **4** | **USB Data Protection Resistor** | `22R` or `33R` (0603 or 0402) | **Signal Noise Suppression:** Connected in series with the D+ and D- lines to suppress pulse noise and signal reflections during USB data transmission. |
| **5** | **USB ESD Protection IC** | `USBLC6-2SC6` (SOT23-6) | **Electrostatic Protection:** Filters and discharges static sparks (caused by cable insertion/removal or human contact) to GND, protecting the USB data lines and preventing chip burnout. |
| **6** | **3.3V Buck Regulator IC (SY8089A)** | `SY8089A` (SOT-23-5) | **Voltage Regulator:** Steps down the 5V input from the USB port to a precise 3.3V to power the entire ESP32-S3 logic system. |
| **7** | **Buck Power Inductor (L1)** | `1uH` to `2.2uH` (High-current power inductor) | **Energy Storage:** A mandatory component of the Buck circuit that converts electrical pulses into a smooth, stable current. |
| **8** | **Power Filtering Capacitors (C3, C4, etc.)** | `10uF` (0603 or 0805) | **Noise Filtering & Energy Storage:** Placed near power input/output terminals and the chip's VDD pin to compensate for sudden current demands during high-load operation. |
| **9** | **Feedback Resistors (R9, R10)** | `450K` and `100K` | **Voltage Setting:** Sets the voltage divider ratio to ensure the Buck IC outputs exactly 3.3V. |
| **10** | **EN / RST Pull-up Resistor** | `10K` (0603 or 0402) | **Logic Level Maintenance:** Keeps the Enable pin at a High level so the chip is always ready for operation, preventing a floating state that could pick up noise. |
| **11** | **Startup Delay Capacitor (C1)** | `1uF` (0603 or 0402) | **RC Delay Circuit:** Ensures the supply voltage is fully stable before allowing the ESP32-S3 chip to boot up. |
| **12** | **Tactile Switch** | `TS-1088-AR02016` | **Manual Reset / Boot:** Allows the user to reset the circuit or manually place the chip into bootloader mode if necessary. |
| **13** | **Antenna Connector (IPEX / U.FL)** | `U.FL-R-SMT` | **RF Transmission:** Connector for an external 2.4GHz antenna (WiFi/BLE) or 4G/GPS antenna to optimize signal range on the drone frame. |
| **14** | **4G Cat 1 Module (SIM7670G)** | `SIM7670G` (LCC+LGA) | **Long-range Communication:** Connects to global LTE Cat 1 cellular networks; handles long-range MAVLink packet transmission for the UAV. |

---

## 🛠️ Design Tools

- **Circuit Design Software:** [EasyEDA](https://easyeda.com/)
- **Component Source:** [LCSC Electronics](https://www.lcsc.com/) (search using the LCSC code in the second column of the table)

---

## 📋 Assembly Notes

- Place power filtering capacitors (`C3`, `C4`, etc.) as close as possible to the VDD pins and the Buck IC input/output terminals to minimize noise.
- Route the antenna (U.FL) away from the drone's carbon fiber frame to prevent signal attenuation and RF interference.
- Carefully check the feedback resistors (`R9`, `R10`) before applying power to ensure the output voltage is exactly 3.3V, preventing damage to the ESP32-S3.
- The SIM767 module can be replaced.
