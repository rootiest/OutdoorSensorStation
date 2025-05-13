# Solar-Powered Sensor Station

## 📚 Table of Contents

- [🧰 Parts List](#-parts-list)
- [ESPHome Configuration](#esphome-configuration)
- [📡 ESPHome Sensor Station Entities](#-esphome-sensor-station-entities)
  - [🌡️ Environmental Sensors](#️-environmental-sensors)
  - [⚡ Power & Energy Monitoring](#-power--energy-monitoring)
  - [📶 WiFi Signal Strength](#-wifi-signal-strength)
  - [🎛️ Switches & Buttons](#-switches--buttons)
- [⚡ Daily Energy Use Summary](#-daily-energy-use-summary)
  - [Base System (5-minute wake cycle)](#base-system-5-minute-wake-cycle)
  - [With PMS5003 Included](#with-pms5003-included)
  - [With OLED Usage (occasional manual activation)](#with-oled-usage-occasional-manual-activation)
- [☀️ Solar Power Summary](#-solar-power-summary)
  - [🔋 + ☀️ Battery Backup & Solar Requirements](#--battery-backup--solar-requirements)
- [ESP32-C3 SuperMini GPIO Pinout Plan](#esp32-c3-supermini-gpio-pinout-plan)
- [💤 Deep Sleep and Power Management](#-deep-sleep-and-power-management)
  - [🔌 Devices Powered Down During Deep Sleep](#-devices-powered-down-during-deep-sleep)
  - [🔓 Wake Lock Behavior](#-wake-lock-behavior)
  - [👆 Display Button Behavior](#-display-button-behavior)
  - [💤 Smart Sleep Behavior](#-smart-sleep-behavior)
    - [Smart Sleep Timing](#smart-sleep-timing)
    - [Preparing for Sleep](#preparing-for-sleep)
    - [Smart PMS Control](#smart-pms-control)
- [🧾 Bill of Materials (BOM Summary)](#-bill-of-materials-bom-summary)
  - [Microcontroller & Power](#microcontroller--power)
  - [Sensors](#sensors)
  - [Display & Interface](#display--interface)
  - [Power Management](#power-management)
  - [Passive Components](#passive-components)
  - [Optional Components](#optional-components)
  - [Downloadable BOM Formats](#downloadable-bom-formats)
- [✅ Conclusion](#-conclusion)

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
| **PI Heating Element (5V, 1W)**            | Maintains safe temp in cold climates   | 0mA (off)            | ≈200mA (when on)             | Controlled via GPIO + MOSFET                         |
| **PMS5003** _(optional)_                   | Air quality sensor                     | ≈1.0mA (sleep)       | 100–120mA                    | Wakes ≈5s per cycle                                  |
| **CN3065**                                 | Solar charge controller                | Negligible           | Passive device               | Handles charging logic                               |
| **5V 200mA Solar Panel**                   | Solar input                            | N/A                  | 200mA max                    | One panel used                                       |
| **3000mAh LiPo Battery**                   | Power storage                          | N/A                  | N/A                          | Powers the full system                               |
| **3× STP55NF06L N-MOSFETs**                | Power switching (OLED, heater)         | 0mA                  | 0mA                          | Logic-level, TO-220, controlled by 3.3V GPIO         |
| **3× 4.7kΩ Resistor (1/4W)**               | Pull-ups for SDA, SCL, and DS18B20     | N/A                  | N/A                          | 2 for I²C (SDA/SCL), 1 for DS18B20                   |
| **100nF Ceramic Capacitor (50V)**          | High-frequency filtering near ESP32    | N/A                  | N/A                          | Place close to ESP32 power pin and sensor power rail |
| **470µF Electrolytic Capacitor (6.3–10V)** | Bulk decoupling                        | N/A                  | N/A                          | Use near ESP32 VIN to buffer inrush currents         |

---

## ESPHome Configuration

[YAML configuration for the Sensor Station](ESPHome/sensor_station.yml)

This will configure the ESPHome firmware for the sensor station.

See [ESPHome documentation](https://esphome.io/) for more details
on how to set up and use ESPHome.

## 📡 ESPHome Sensor Station Entities

### 🌡️ Environmental Sensors

- Ambient Light
- Ambient Temperature
- Ambient Humidity
- BMP280 Temperature
- Atmospheric Pressure
- Enclosure Internal Temp
- Pond Water Temperature
- Water Level Distance

### ⚡ Power & Energy Monitoring

- Estimated Total Power Use
- Battery Voltage
- Battery Current
- Battery Power
- Solar Voltage
- Solar Current
- Solar Power
- Total Energy Used Today (daily cumulative)
- Total Energy Used (All-Time) (lifetime cumulative)
- Solar Energy Collected Today (daily cumulative)

### 📶 WiFi Signal Strength

- WiFi Signal dB (signal strength in decibels)
- WiFi Signal Percent (signal strength percentage)

### 🎛️ Switches & Buttons

- OLED Display Activation (button to activate OLED display temporarily)
- PTC Heater Switch (switch to control enclosure heater)

---

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

## ESP32-C3 SuperMini GPIO Pinout Plan

| Purpose                        | GPIO   | Wire Color          | Notes                                             |
| ------------------------------ | ------ | ------------------- | ------------------------------------------------- |
| **5V**                         | ----   | Red                 | 5V Rail                                           |
| **GND**                        | ----   | Black               | Ground Plane                                      |
| **3V3**                        | ----   | Red w/ stripe       | 3.3V Rail                                         |
| **I²C SDA (AHT20/BMP280)**     | GPIO4  | White               | Shared I²C bus                                    |
| **I²C SCL (AHT20/BMP280)**     | GPIO5  | Yellow w/ stripe    | Shared I²C bus                                    |
| **1-Wire (DS18B20)**           | GPIO6  | Blue /w stripe      | Needs 4.7kΩ pull-up resistor                      |
| **Heater Power Switch**        | GPIO2  | Yellow              | Drives N-MOSFET to power heater                   |
| **HC-SR04 Trigger**            | GPIO10 | Brown w/ stripe     | Fires Ultrasonic Output                           |
| **HC-SR04 Echo**               | GPIO9  | White w/ stripe     | Receives Ultrasonic Input                         |
| **PMS5003 TX → RX**            | GPIO21 | Yellow w/ 2 stripes | UART receive from PMS5003                         |
| **PMS5003 RX ← TX**            | GPIO20 | Green w/ stripe     | UART send to PMS5003                              |
| **PMS Power Switch**           | GPIO8  | White w/ 2 stripes  | Drives N-MOSFET to power PMS                      |
| **TEMT6000 (Analog Out)**      | GPIO0  | Green               | Analog read (ADC1), safe and accessible           |
| **OLED Power (MOSFET Gate)**   | GPIO7  | Brown               | Drives N-MOSFET to power OLED                     |
| **OLED Button Input**          | GPIO1  | Blue                | Safe as input only; must be HIGH or float at boot |
| **Sensor Power (MOSFET Gate)** | GPIO3  | Yellow              | Drives N-MOSFET to power I²C and Ultrasonic       |

These color assignments are not electrically required but help reduce
wiring mistakes during assembly.

---

## 💤 Deep Sleep and Power Management

The ESP32-C3 enters **deep sleep** between sensor readings to conserve energy.  
In this state, the main processor halts and only the RTC (real-time clock)
memory and wake-up timer remain powered.  
This reduces current consumption to as low as **5µA**,
making deep sleep essential for maintaining long battery life in
solar-powered deployments.

During deep sleep, most GPIO pins are inactive, and peripheral
devices are unpowered unless explicitly designed to stay on.

### 🔌 Devices Powered Down During Deep Sleep

The following components are fully powered off
while the ESP32-C3 is in deep sleep:

- **SH1107 OLED Display**  
  Powered via a GPIO-controlled MOSFET, only powers on with button press
- **AHT20 + BMP280 Combo Sensor**  
  On the I²C sensor power rail
- **TMP102 Temperature Sensor**  
  Shares the I²C power rail
- **TEMT6000 Light Sensor**  
  Powered via sensor rail, inactive during sleep
- **HC-SR04 Ultrasonic Sensor**  
  Powered by sensor rail, to avoid idle current
- **PMS5003 Air Quality Sensor**  
  Powered via a GPIO-controlled MOSFET
- **DS18B20 Temperature Probe**  
  Powered off via sensor rail
- **PI Heating Element**  
   Powered via a GPIO-controlled MOSFET, not energized except
  during extreme cold during wakes
- **Wi-Fi Radio**  
  Disabled automatically in deep sleep mode

Only the **battery-backed RTC** and **wake timer** remain active.

### 🔓 Wake Lock Behavior

A **wake lock** can temporarily prevent the ESP32-C3 from
returning to deep sleep.  
This mechanism is useful for debugging, testing, or when extended uptime
is required (e.g. for screen visibility or serial logs).

When a wake lock is active, the OLED display shows a clear full-screen message:

```
WAKE
LOCK
```

This mode takes priority over the normal page-based display logic,
ensuring the user is alerted whenever the wake lock is engaged.  
The message uses a large font and occupies the full screen for visibility.

### 👆 Display Button Behavior

The OLED remains off by default and only lights up on button press.  
It is automatically turned off after a brief timeout to preserve energy.

### 💤 Smart Sleep Behavior

The system uses a script-based approach to determine when and how long to enter
deep sleep, with logic based on wake-lock state,
battery voltage, and solar input.

#### Wake Timing

An initial delay of 30s occurs at the start of wake/boot
when the PMS5003 is enabled.  
This allows the PMS5003 to stabilize before taking readings.

After the initial wake-up (or boot-up) the smart sleep script is executed.

#### Smart Sleep Timing

This script dynamically chooses a sleep duration
based on current power conditions:

- If `wake_hold_switch` is **on**,
  the system logs the event and **skips sleep**.
- Otherwise, it evaluates:
  - If `force_long_sleep` is **on**,
    the system enters **long sleep** (30 minutes).
  - Else:
    - If battery voltage is **above 3.7V**
      and solar input is **above 50 mA**,
      it chooses a **short sleep** duration (5 minutes).
    - If not, it defaults to **long sleep** (30 minutes).
- After setting the sleep duration,
  the system **remains awake for a short time**
  to allow sensors to stabilize before sleeping:
  - If the PMS5003 air sensor is powered, it waits **30 seconds**.
  - Otherwise, it waits **10 seconds**.
- If wake-lock is **not active**,
  the script initiates deep sleep after the delay.

---

#### Preparing for Sleep

This script ensures all major components are powered down
before entering deep sleep.

Actions:

- Logs the transition to sleep.
- Powers off:
  - OLED display
  - PMS5003 sensor
  - Shared sensor rail
  - Internal heater
- Delays 100ms for power-down settling.
- Initiates deep sleep.

---

#### Smart PMS Control

The PMS5003 is relatively power-hungry, so it is only powered on
when conditions are favorable.  
The PMS5003 uses ~80–100 mA when measuring and ~20–30 mA in its idle state.

This script decides whether to power the PMS5003 sensor
based on power availability.

- If battery is **above 3.6V** or solar current is **above 20 mA**,
  the PMS sensor is powered on.
- Otherwise, it remains off to conserve energy.

---

## 🧾 Bill of Materials (BOM Summary)

### Microcontroller & Power

- 1× ESP32-C3 SuperMini
- 1× CN3065 Solar Charging Module
- 1× 3000mAh LiPo Battery
- 1× 5V 200mA Solar Panel

### Sensors

- 1× AHT20 + BMP280 (combined I²C PCB)
- 1× TMP102 (I²C internal temp)
- 1× DS18B20 (1-Wire, waterproof)
- 1× TEMT6000 (analog light sensor)
- 1× HC-SR04 (ultrasonic distance)
- 2× INA219 (voltage/current monitoring)
- 1× PMS5003 (air quality sensor, optional)

### Display & Interface

- 1× SH1107 OLED Display (128x128 I²C)
- 1× Momentary button (for OLED activation)

### Power Management

- 1× PI Heating Element (5V, 1W)
- 3× STP55NF06L N-MOSFETs (OLED + Heater switching)

#### Passive Components

- 3× 4.7kΩ Resistors (1/4W)
  - 2 for I²C pull-ups (SDA + SCL shared across all I²C devices)
  - 1 for DS18B20 1-Wire pull-up
- 1× 470µF Electrolytic Capacitor (≥6.3V) – Bulk decoupling (for ESP32 VIN)
- 1× 100nF Ceramic Capacitor (≥50V) – High-frequency filtering
  (ESP32 + sensor power rails)

### Optional Components

- Assorted hookup wires (solid or stranded) for interconnections
- Custom PCB or perfboard for permanent soldered assembly
- Connectors (e.g. JST-PH, JST-XH, Dupont, terminal blocks) for modular assembly
- Heat-shrink tubing or silicone wire sleeving for strain relief
- Mounting hardware or standoffs for enclosure assembly
- 1× 3D-printed or waterproof case
- 1× Heatsink pad or thermal tape (for PTC heater to LiPo)
- 1× GPIO expander (not needed in current design)

---

### Downloadable BOM Formats

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

```

```
