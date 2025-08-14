# Real-Time Embedded Data Logger (ESP32 + FreeRTOS)

A production-ready ESP32-based **real-time data logger** that samples **temperature & humidity (DHT11)**, **barometric pressure (BMP180/BMP085)**, timestamps data via **DS3231 RTC**, and stores records on an **SD card**—all orchestrated with **FreeRTOS tasks** and an inter-task **queue**.

> Built from Prashant Malpure's project code and structured for GitHub with clear documentation, wiring, and setup instructions.

## Features
- FreeRTOS **two-task** architecture:
  - `SensorTask` reads DHT11 + BMP180/BMP085, and DS3231 time
  - `SDCardTask` appends records to `/datalog.txt` on the SD card
- **Queue-based decoupling** between sensing and storage for robustness
- **Retry** logic for SD open failures, with status logs over Serial
- CSV-like log line with date, time, temperature (°C), humidity (%), pressure (hPa)
- Designed for **ESP32** (Arduino core); portable to PlatformIO

## Hardware
- ESP32 DevKit (tested on generic WROOM dev board)
- DHT11 (temp & humidity)
- BMP180/BMP085 (pressure)
- DS3231 RTC module
- MicroSD card module
- Jumper wires, breadboard

### Default Pins
| Peripheral | ESP32 Pin | Notes |
|---|---:|---|
| DHT11 data | **GPIO 4** | `#define DHTPIN 4` |
| SD CS      | **GPIO 5** | `#define SD_CS 5` |
| I2C SDA    | **GPIO 21** | `Wire.begin(21, 22)` |
| I2C SCL    | **GPIO 22** |  |

> Change pins in `src/main.ino` if your hardware differs.

## Wiring (Quick Guide)
- **DHT11**: VCC → 3V3, GND → GND, DATA → GPIO 4 (w/ 10k pull-up recommended)
- **BMP180/BMP085** (I2C): VCC → 3V3, GND → GND, SDA → GPIO 21, SCL → GPIO 22
- **DS3231** (I2C): VCC → 3V3, GND → GND, SDA → GPIO 21, SCL → GPIO 22
- **SD Module** (SPI): CS → GPIO 5, SCK → GPIO 18, MISO → GPIO 19, MOSI → GPIO 23, VCC → 3V3/5V*, GND → GND  
  *Use a 3.3V-compatible SD module or level shifting for 5V modules.*

## Library Dependencies (Arduino IDE → Manage Libraries)
- **RTClib** (Adafruit)
- **DHT sensor library** (Adafruit) + **Adafruit Unified Sensor**
- **Adafruit BMP085 Unified** (works with BMP180)
- **SD** (built-in ESP32 Arduino core)
- **FS**, **SPI**, **Wire** (built-in)

## Build & Upload

### Arduino IDE
1. Install **ESP32 boards** in Boards Manager (Espressif Systems).
2. Board: `ESP32 Dev Module`. Flash freq: 40/80MHz, Partition: Default.
3. Install the libraries above.
4. Open `src/main.ino` and **Upload**.

### PlatformIO (VS Code)
```ini
; platformio.ini (example)
[env:esp32dev]
platform = espressif32
board = esp32dev
framework = arduino
monitor_speed = 115200
lib_deps =
  adafruit/RTClib
  adafruit/DHT sensor library
  adafruit/Adafruit Unified Sensor
  adafruit/Adafruit BMP085 Unified
```
- Place code from `src/` as your environment's source. Build → Upload.

## First-Time RTC Set
If your DS3231 time is not set, upload `src/rtc_set_time.ino` **once** to set the RTC to the compile time. After it runs, re-flash `main.ino`.

## Log Sample
```
14-08-2025, 12:00:05, Temperature: 29.2 °C, Humidity: 58.0 %, Pressure: 1008.6 hPa
```

## Project Structure
```
real-time-embedded-data-logger/
├─ src/
│  ├─ main.ino              # FreeRTOS tasks + logging
│  └─ rtc_set_time.ino      # One-time RTC time setter
├─ docs/
│  └─ wiring-notes.md       # Pinout & hookup notes
├─ .gitignore
├─ LICENSE
└─ README.md
```

## How It Works
- `SensorTask` (higher priority) reads sensors every **5 s**, formats a line, and pushes it to a **queue**.
- `SDCardTask` pops from the queue and appends to **/datalog.txt**. It retries opening the file (3x) before reporting failure.
- `vTaskDelay` yields CPU to other tasks; no work in `loop()` (FreeRTOS scheduler runs tasks).

## Troubleshooting
- **SD init failed**: Check CS pin, wiring, and that SD is FAT32.
- **DHT read fail**: Ensure stable power & pull-up; DHT11 is slow—keep ≥1–2s interval.
- **No RTC/I2C**: Verify SDA/SCL pins and module power.
- **Garbled Serial**: Match `115200` baud in Serial Monitor.

## License
MIT — see `LICENSE`.

---

Authored and organized by **Prashant Ananda Malpure**.
