# ESP32-C3 Sensor Station Parts List & Energy Summary

## 🧰 Parts List

| Component                | Function                                       | Idle Current         | Active Current               | Notes                                                            |
| ------------------------ | ---------------------------------------------- | -------------------- | ---------------------------- | ---------------------------------------------------------------- |
| **ESP32-C3 SuperMini**   | Main MCU, Wi-Fi                                | ~0.01mA (deep sleep) | ~80–120mA (peak during WiFi) | ~20–30mA avg over wake time                                      |
| **AHT20**                | Temp + Humidity                                | ~0.5mA               | ~1.0mA                       | Very low power                                                   |
| **BMP280**               | Pressure                                       | ~0.5mA               | ~0.7mA                       | Combined board w/ AHT20                                          |
| **SH1107 OLED (128x128)**| Visual output (I²C)                            | ~0.5mA (off or idle) | ~10–15mA (typical on)        | Display will be turned on briefly with button press              |
| **TEMT6000**             | Ambient Light                                  | 0mA (powered off)    | ~0.5–1mA                     | Simple analog sensor                                             |
| **HC-SR04**              | Ultrasonic distance                            | 0mA (powered off)    | ~15–20mA                     | Only on briefly to take reading                                  |
| **2× INA219**            | Current/voltage monitor                        | ~0.6mA each          | ~1.0mA each                  | Continuously powered                                             |
| **DS18B20**              | Waterproof temp (fish pond)                    | ~0.5mA               | ~1.5mA                       | Needs pull-up resistor                                           |
| **PMS5003** *(optional)* | Air quality sensor                             | ~1.0mA (sleep)       | 100–120mA                    | Wakes ~5s per cycle                                              |
| **CN3065**               | Solar charge controller                        | Negligible           | Passive device               | Handles charging logic                                           |
| **5V 200mA Solar Panel** | Solar input                                    | N/A                  | 200mA max                    | One panel used                                                   |
| **3000mAh LiPo Battery** | Power storage                                  | N/A                  | N/A                          | Powers the full system                                           |
| **Resistor Kit**         | Pull-ups, misc use                             | N/A                  | N/A                          | 4.7kΩ used for I²C and DS18B20                                   |
| **Capacitor Kit**        | Power stabilization (inrush current buffering) | N/A                  | N/A                          | Add 470–1000µF cap near ESP32 VIN; optional 0.1µF ceramic nearby |

---

## ⚡ Daily Energy Use Summary

### Base System (5-minute wake cycle)

- **ESP32-C3 (deep sleep)**: ~0.01mA × 23.6h ≈ **0.24mAh/day**
- **ESP32-C3 (active)**: ~25mA avg × 5s per wake × 288 wakes = **10.0mAh/day**
- **Sensors (active during wake)**:
  - AHT20/BMP280, HC-SR04, DS18B20, INA219s, etc. ≈ **15mA combined** × 5s × 288 = **6.0mAh/day**
- **TEMT6000 (brief read)**: negligible (included above)
- **OLED (off by default)**: **0mAh/day** (fully unpowered unless activated manually)

- **Subtotal without PMS5003 or OLED usage**:  
  → **~16.2mAh/day**

---

### With PMS5003 Included

- **PMS5003 sleep**: ~1mA × 24h = **24mAh/day**
- **PMS5003 active**: ~100mA × 5s × 288 = **40mAh/day**
- **New total with PMS5003**:  
  → **~80.2mAh/day**

---

### With OLED Usage (occasional manual activation)

Assuming:
- OLED on for 15s × 10 button presses per day
- OLED draw ≈ 12mA

→ 12mA × (15s × 10) / 3600s = **~0.5mAh/day**

---

### 📊 Final Estimates

| Configuration           | Estimated Daily Draw |
| ----------------------- | -------------------- |
| Base system only        | ~16mAh/day           |
| With OLED use           | ~16.5mAh/day         |
| With PMS5003            | ~80mAh/day           |
| With PMS5003 + OLED use | ~80.5mAh/day         |



---

## 🔋 Battery Backup Estimates

| Scenario           | Daily Usage | Backup Days (3000mAh) |
|--------------------|-------------|------------------------|
| Without PMS5003    | ~85mAh      | ~35 days               |
| With PMS5003       | ~117mAh     | ~25 days               |

---

## ☀️ Solar Power Summary

- **Panel Output**: 5V @ 200mA
- **Estimated good sunlight**: 4h/day
- **Daily energy input**: ~800mAh raw, ~400mAh effective
- **Conclusion**: More than sufficient to offset daily usage, even in winter/cloudy days

---
### ESP32-C3 SuperMini GPIO Pinout Plan

| Purpose                      | GPIO   | Notes                                                    |
| ---------------------------- | ------ | -------------------------------------------------------- |
| **5V**                       | ----   | 5V Rail                                                  |
| **GND**                      | ----   | Ground Plane                                             |
| **3V3**                      | ----   | 3.3V Rail                                                |
| **I²C SDA (AHT20/BMP280)**   | GPIO4  | Shared I²C bus                                           |
| **I²C SCL (AHT20/BMP280)**   | GPIO5  | Shared I²C bus                                           |
| **1-Wire (DS18B20)**         | GPIO6  | Needs 4.7kΩ pull-up resistor                             |
| **Sensor power switch**      | GPIO2  | MOSFET-controlled rail for sensors                       |
| **HC-SR04 Trigger**          | GPIO10 | -                                                        |
| **HC-SR04 Echo**             | GPIO3  | -                                                        |
| **PMS5003 TX → RX**          | GPIO21 | UART receive from PMS5003                                |
| **PMS5003 RX ← TX**          | GPIO20 | UART send to PMS5003                                     |
| **PMS5003 SET (Sleep Ctrl)** | GPIO1  | Drive LOW to enter PMS sleep mode                        |
| **TEMT6000 (Analog Out)**    | GPIO0  | Analog read (ADC1), safe and accessible                  |
| **OLED Power (MOSFET Gate)** | GPIO7  | Drives N-MOSFET to power OLED                            |
| **OLED Button Input**        | GPIO9  | Safe as input only; must be HIGH or float at boot        |
| **UNUSED**                   | GPIO8  | Safe as input only; must be HIGH at reset                |

---

## ✅ Conclusion

- System is **highly energy efficient**
- A **single 5V 200mA panel** and **3000mAh battery** are **more than enough**
- Even with PMS5003, the design remains **solar-sustainable**
- Power draw is **well within ESP32-C3 GPIO limits**, so **no external MOSFETs required**
