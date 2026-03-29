# Arduino Cooperative Task Scheduler with Environmental Monitoring

> A non-blocking, cooperative task scheduler running 4 concurrent time-critical tasks on an Arduino Uno — with real-time DHT22 sensor interfacing, I2C LCD display, adaptive threshold alerting, and audio feedback via piezo buzzer.

---

## Demo

> Simulated on [Wokwi](https://wokwi.com) — no hardware required to run.

---

## Features

- **Cooperative Task Scheduler** — 4 independent tasks running concurrently using `millis()`-based non-blocking timing. Zero use of `delay()` after startup.
- **DHT22 Sensor Interfacing** — Real-time temperature and humidity acquisition every 2 seconds.
- **Heat Index Computation** — Derived from temperature and humidity using the Steadman formula via the Adafruit DHT library.
- **Adaptive Min/Max Auto-Calibration** — Automatically tracks observed minimum and maximum values for both temperature and humidity across the session.
- **Threshold-Based Alerting** — Triggers when temperature exceeds 30 °C or humidity exceeds 70%, with three distinct buzzer tones based on alert condition.
- **16×2 I2C LCD Display** — Live readout of temperature, humidity, heat index, and alert status refreshed every second.
- **Serial Monitor Output** — Structured data log including observed range and alert status, printed every 2 seconds.

---

## Hardware / Components

| Component | Quantity |
|---|---|
| Arduino Uno | 1 |
| DHT22 Temperature & Humidity Sensor | 1 |
| 16×2 LCD with I2C module (PCF8574) | 1 |
| Piezo Buzzer | 1 |
| LED (any colour) | 1 |
| 220Ω Resistor | 1 |
| Breadboard + jumper wires | — |

---

## Wiring

### DHT22
| DHT22 Pin | Arduino |
|---|---|
| VCC | 5V |
| DATA | Digital pin 2 |
| GND | GND |

### LED
| | Arduino |
|---|---|
| Anode (+) → 220Ω resistor | Digital pin 13 |
| Cathode (−) | GND |

### Piezo Buzzer
| Buzzer | Arduino |
|---|---|
| + (positive) | Digital pin 8 |
| − (negative) | GND |

### LCD I2C
| LCD I2C Pin | Arduino |
|---|---|
| VCC | 5V |
| GND | GND |
| SDA | A4 |
| SCL | A5 |

---

## Task Scheduler Design

The core of this project is a cooperative scheduler implemented entirely in software — no RTOS, no interrupts.

```
loop() runs continuously
│
├── taskSensorRead()   — executes every 2000 ms
├── taskAlertCheck()   — executes every  500 ms
├── taskLCD()          — executes every 1000 ms
└── taskSerial()       — executes every 2000 ms
```

Each task follows the same non-blocking pattern:

```cpp
void taskX() {
  if (millis() - lastRunX < INTERVAL_X) return;
  lastRunX = millis();
  // task logic here
}
```

This means the CPU never blocks — it returns to `loop()` immediately if a task is not due, making all four tasks appear to run simultaneously.

### Shared Memory

All tasks communicate through a single `struct SensorData`:

```cpp
struct SensorData {
  float temperature;
  float humidity;
  float heatIndex;
  float minTemp, maxTemp;
  float minHumidity, maxHumidity;
  bool  alertActive;
};
```

Task 1 writes sensor values → Tasks 2, 3, 4 read them. No locking needed since Arduino is single-core and tasks are cooperative (not preemptive).

---

## Alert Logic

| Condition | LED | Buzzer |
|---|---|---|
| All normal | Off | Silent |
| Temp > 30 °C only | Blinking | 900 Hz beep every 500 ms |
| Humidity > 70% only | Blinking | 600 Hz beep every 500 ms |
| Both exceeded | Blinking | 1200 Hz beep every 500 ms |

---

## LCD Display Layout

```
Row 0:  T:28.4C  H:65.2%
Row 1:  HI:29.1C OK
```

On alert:
```
Row 0:  T:32.5C  H:75.3%
Row 1:  HI:36.1C !!ALERT!!
```

---

## Serial Monitor Output

```
===== Environmental Monitor =====
Temperature : 25.0 C
Humidity    : 60.0 %
Heat Index  : 25.8 C
------- Observed Range ----------
Temp  min/max: 25.0 / 32.5 C
Hum   min/max: 60.0 / 75.3 %
Alert Status: OK
=================================
```

---

## Libraries Required

| Library | Install via |
|---|---|
| DHT sensor library (Adafruit) | Arduino Library Manager |
| Adafruit Unified Sensor | Arduino Library Manager |
| LiquidCrystal I2C (Frank de Brabander) | Arduino Library Manager |

In Arduino IDE: `Sketch → Include Library → Manage Libraries` → search each name above.

---

## How to Run (Wokwi Simulator)

1. Go to [wokwi.com](https://wokwi.com) → New Project → Arduino Uno
2. Add components: DHT22, LCD1602 I2C, Piezo Buzzer, LED, 220Ω resistor
3. Wire as per the table above
4. Paste `src/main.ino` into the editor
5. Click ▶ Play → open Serial Monitor at 9600 baud
6. Click the DHT22 and drag the temperature slider above 30 °C to trigger an alert

## How to Run (Real Hardware)

1. Wire components to Arduino Uno as per the wiring table
2. Open `src/main.ino` in Arduino IDE
3. Install the three libraries listed above
4. Select `Tools → Board → Arduino Uno` and the correct Port
5. Click Upload

If the LCD stays blank, change `0x27` to `0x3F` in the `LiquidCrystal_I2C lcd(...)` line — some I2C modules use a different address.

---

## Concepts Demonstrated

- Cooperative task scheduling without an RTOS
- Non-blocking timing using `millis()`
- Shared memory across tasks via a global struct
- Real-time sensor interfacing (DHT22)
- Heat index computation
- I2C communication protocol (LCD)
- Adaptive auto-calibration (min/max tracking)
- Multi-condition threshold alerting with audio feedback

---

## Project Structure

```
arduino-cooperative-task-scheduler/
│
├── src/
│   └── main.ino
│
├── images/
│   └── wokwi_circuit.png
│
└── README.md
```

---

## License

MIT License — free to use, modify, and distribute.
