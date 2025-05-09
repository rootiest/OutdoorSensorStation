# Solar-Powered Sensor Station

## 🧰 Parts List

| Component                                  | Function                               | Idle Current         | Active Current               | Notes                                                |
| ------------------------------------------ | -------------------------------------- | -------------------- | ---------------------------- | ---------------------------------------------------- |
| **ESP32-C3 SuperMini**                     | Main MCU, Wi-Fi                        | ≈0.01mA (deep sleep) | ≈80–120mA (peak during WiFi) | ≈20–30mA avg over wake time                          |
| **AHT20**                                  | Temp + Humidity                        | ≈0.5mA               | ≈1.0mA                       | Very low power                                       |
| **BMP280**                                 | Pressure                               | ≈0.5mA               | ≈0.7mA                       | Combined board w/ AHT20                              |
| **TMP102**                                 | Internal enclosure temperature monitor | ≈10–50μA             | ≈100μA max                   | I²C-based, very low-power                            |
| **SH1107 OLED (128x128)**                  | Visual output (I²C)                    | ≈0.5mA (off or idle) | ≈10–15mA (typical on)        | Display will be turned on briefly with button press  |
| **TEMT6000**                               | Ambient Light                          | 0mA (powered off)    | ≈0.5–1mA                     | Simple analog sensor                                 |
| **HC-SR04**                                | Ultrasonic distance                    | 0mA (powered off)    | ≈15–20mA                     | Only on briefly to take reading                      |
| **2× INA219**                              | Current/voltage monitor                | ≈0.6mA each          | ≈1.0mA each                  | Continuously powered                                 |
| **DS18B20**                                | Waterproof temp (fish pond)            | ≈0.5mA               | ≈1.5mA                       | Needs pull-up resistor                               |
| **PTC Heater (5V, 1W)**                    | Maintains safe temp in cold climates   | 0mA (off)            | ≈200mA (when on)             | Controlled via GPIO + MOSFET                         |
| **PMS5003** _(optional)_                   | Air quality sensor                     | ≈1.0mA (sleep)       | 100–120mA                    | Wakes ≈5s per cycle                                  |
| **CN3065**                                 | Solar charge controller                | Negligible           | Passive device               | Handles charging logic                               |
| **5V 200mA Solar Panel**                   | Solar input                            | N/A                  | 200mA max                    | One panel used                                       |
| **3000mAh LiPo Battery**                   | Power storage                          | N/A                  | N/A                          | Powers the full system                               |
| **2× STP55NF06L N-MOSFETs**                | Power switching (OLED, heater)         | 0mA                  | 0mA                          | Logic-level, TO-220, controlled by 3.3V GPIO         |
| **3× 4.7kΩ Resistor (1/4W)**               | Pull-ups for SDA, SCL, and DS18B20     | N/A                  | N/A                          | 2 for I²C (SDA/SCL), 1 for DS18B20                   |
| **100nF Ceramic Capacitor (50V)**          | High-frequency filtering near ESP32    | N/A                  | N/A                          | Place close to ESP32 power pin and sensor power rail |
| **470µF Electrolytic Capacitor (6.3–10V)** | Bulk decoupling                        | N/A                  | N/A                          | Use near ESP32 VIN to buffer inrush currents         |

---

## ESPHome Configuration

[YAML configuration for the Sensor Station](ESPHome/sensor_station.yml)

This will configure the ESPHome firmware for the sensor station.

See [ESPHome documentation](https://esphome.io/) for more details
on how to set up and use ESPHome.

## ⚡ Daily Energy Use Summary

### Base System (5-minute wake cycle)

- **ESP32-C3 (deep sleep)**: ≈0.01mA × 23.6h ≈ **0.24mAh/day**
- **ESP32-C3 (active)**: ≈25mA avg × 5s per wake × 288 wakes = **10.0mAh/day**
- **Sensors (active during wake)**:
  - AHT20/BMP280, HC-SR04, DS18B20, INA219s, TMP102, etc.  
    ≈ **15.05mA combined** × 5s × 288 = **6.01mAh/day**
- **TEMT6000 (brief read)**: negligible (included above)
- **OLED (off by default)**: **0mAh/day** (fully unpowered unless activated manually)

- **Subtotal without PMS5003 or OLED usage**:  
  → **≈16.25mAh/day**

---

### With PMS5003 Included

- **PMS5003 sleep**: ≈1mA × 24h = **24mAh/day**
- **PMS5003 active**: ≈100mA × 5s × 288 = **40mAh/day**
- **New total with PMS5003**:  
  → **≈80.05mAh/day**

---

### With OLED Usage (occasional manual activation)

Assuming:

- OLED on for 15s × 10 button presses per day
- OLED draw ≈ 12mA

→ 12mA × (15s × 10) / 3600s = **≈0.5mAh/day**

- **Base system + TMP102 + OLED use**:  
  → **≈16.75mAh/day**

---

## ☀️ Solar Power Summary

- **Panel Output**: 5V @ 200mA
- **Estimated good sunlight**: 4h/day
- **Daily energy input**: ≈800mAh raw, ≈400mAh effective
- **Conclusion**: More than sufficient to offset daily usage,
  even in winter/cloudy days

---

### 🔋 + ☀️ Battery Backup & Solar Requirements

With 3000mAh LiPo and 5V 200mA Solar Panel

| Scenario                | Daily Draw | Backup Days | Solar Required | Sun Hours Needed   |
| ----------------------- | ---------- | ----------- | -------------- | ------------------ |
| Base system only        | ≈16.25mAh  | ≈184 days   | ≈16.25mAh      | ≈0.08 hr (≈5 min)  |
| With OLED use           | ≈16.75mAh  | ≈179 days   | ≈16.75mAh      | ≈0.08 hr           |
| With PMS5003            | ≈80.05mAh  | ≈37 days    | ≈80.05mAh      | ≈0.40 hr (≈24 min) |
| PMS5003 + OLED use      | ≈80.55mAh  | ≈37 days    | ≈80.55mAh      | ≈0.40 hr           |
| Base system + heater    | ≈216.25mAh | ≈13.9 days  | ≈216.25mAh     | ≈1.08 hr           |
| OLED + heater           | ≈216.75mAh | ≈13.8 days  | ≈216.75mAh     | ≈1.08 hr           |
| PMS5003 + heater        | ≈280.05mAh | ≈10.7 days  | ≈280.05mAh     | ≈1.40 hr           |
| PMS5003 + OLED + heater | ≈280.55mAh | ≈10.7 days  | ≈280.55mAh     | ≈1.40 hr           |

---

### ESP32-C3 SuperMini GPIO Pinout Plan

| Purpose                      | GPIO   | Notes                                             |
| ---------------------------- | ------ | ------------------------------------------------- |
| **5V**                       | ----   | 5V Rail                                           |
| **GND**                      | ----   | Ground Plane                                      |
| **3V3**                      | ----   | 3.3V Rail                                         |
| **I²C SDA (AHT20/BMP280)**   | GPIO4  | Shared I²C bus                                    |
| **I²C SCL (AHT20/BMP280)**   | GPIO5  | Shared I²C bus                                    |
| **1-Wire (DS18B20)**         | GPIO6  | Needs 4.7kΩ pull-up resistor                      |
| **Sensor power switch**      | GPIO2  | MOSFET-controlled rail for sensors                |
| **HC-SR04 Trigger**          | GPIO10 | -                                                 |
| **HC-SR04 Echo**             | GPIO3  | -                                                 |
| **PMS5003 TX → RX**          | GPIO21 | UART receive from PMS5003                         |
| **PMS5003 RX ← TX**          | GPIO20 | UART send to PMS5003                              |
| **PMS5003 SET (Sleep Ctrl)** | GPIO1  | Drive LOW to enter PMS sleep mode                 |
| **TEMT6000 (Analog Out)**    | GPIO0  | Analog read (ADC1), safe and accessible           |
| **OLED Power (MOSFET Gate)** | GPIO7  | Drives N-MOSFET to power OLED                     |
| **OLED Button Input**        | GPIO9  | Safe as input only; must be HIGH or float at boot |
| **UNUSED**                   | GPIO8  | Safe as input only; must be HIGH at reset         |

---

### 🧾 Bill of Materials (BOM Summary)

#### Microcontroller & Power

- 1× ESP32-C3 SuperMini
- 1× CN3065 Solar Charging Module
- 1× 3000mAh LiPo Battery
- 1× 5V 200mA Solar Panel

#### Sensors

- 1× AHT20 + BMP280 (combined I²C PCB)
- 1× TMP102 (I²C internal temp)
- 1× DS18B20 (1-Wire, waterproof)
- 1× TEMT6000 (analog light sensor)
- 1× HC-SR04 (ultrasonic distance)
- 2× INA219 (voltage/current monitoring)
- 1× PMS5003 (air quality sensor, optional)

#### Display & Interface

- 1× SH1107 OLED Display (128x128 I²C)
- 1× Momentary button (for OLED activation)

#### Power Management

- 1× PTC Heater (5V, 1W)
- 2× STP55NF06L N-MOSFETs (OLED + Heater switching)

#### Passive Components

- 3× 4.7kΩ Resistors (1/4W)
  - 2 for I²C pull-ups (SDA + SCL shared across all I²C devices)
  - 1 for DS18B20 1-Wire pull-up
- 1× 470µF Electrolytic Capacitor (≥6.3V) – Bulk decoupling (for ESP32 VIN)
- 1× 100nF Ceramic Capacitor (≥50V) – High-frequency filtering
  (ESP32 + sensor power rails)

#### Optional Components

- Assorted hookup wires (solid or stranded) for interconnections
- Custom PCB or perfboard for permanent soldered assembly
- Connectors (e.g. JST-PH, JST-XH, Dupont, terminal blocks) for modular assembly
- Heat-shrink tubing or silicone wire sleeving for strain relief
- Mounting hardware or standoffs for enclosure assembly
- 1× 3D-printed or waterproof case
- 1× Heatsink pad or thermal tape (for PTC heater to LiPo)
- 1× GPIO expander (not needed in current design)

---

#### Downloadable BOM Formats

- [CSV](BOM/sensor_station_bom.csv)
- [JSON](BOM/sensor_station_bom.json)

## ✅ Conclusion

- The system is **extremely energy efficient** in typical operation
  (≈16–17mAh/day), easily supported by a 3000mAh battery.
- A **single 5V 200mA solar panel** can fully offset even the
  highest power use case (≈280mAh/day) with just **1.5 hours of sunlight/day**.
- The design remains **solar-sustainable year-round**, including
  during **cold winter conditions** with heater usage.
- A **low-power PTC heater** and internal **TMP102 temperature sensor**
  allow safe LiPo and PMS5003 operation below freezing.
- The OLED display is manually activated and powered via MOSFET,
  avoiding idle drain and preserving battery life.
- All components are powered efficiently via GPIO-controlled switches,
  using **MOSFETs** to manage loads above safe GPIO sourcing limits.
- All necessary functionality—including sensors,
  power switches, UART, and buttons—was
  achieved using the ESP32-C3 SuperMini's onboard GPIOs,
  with **no GPIO expander required**.
