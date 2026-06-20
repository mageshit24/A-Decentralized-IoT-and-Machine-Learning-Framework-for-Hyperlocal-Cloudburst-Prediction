# 🌩️ Decentralized IoT & ML Framework for Hyperlocal Cloudburst Prediction

A low-cost, decentralized early-warning system that combines **edge sensing (ESP32 + LoRa)**, **machine learning (Random Forest)**, and **real-time visualization (Flask + MySQL)** to predict hyperlocal cloudburst / landslide risk and alert affected areas before disaster strikes.

> 🎓 Final Year Major Project

---

## 📖 Overview

Traditional weather forecasting operates at a regional scale and often misses **hyperlocal** triggers for sudden cloudbursts and landslides — rapid spikes in soil moisture, rainfall intensity, and humidity in a specific micro-zone (a hillside, a single village, a slope).

This project deploys low-power **ESP32 sensor nodes** in the field to continuously monitor environmental conditions. Sensor readings are transmitted through **two independent, decentralized channels**:

1. **LoRa (long-range radio)** → to a local receiver node for on-site alerting (LCD display, buzzer, Blynk cloud dashboard, GSM SMS alerts) — works even without WiFi/internet at the sensor site.
2. **WiFi (UDP)** → directly to a central Flask server, which runs the trained ML model, stores results in MySQL, and serves a live web dashboard.

This dual-path design means the system keeps working locally (buzzer/SMS/LCD) even if the network/internet connection to the central server is down — a key requirement for disaster-prone, connectivity-poor regions.

---

## 🏗️ System Architecture

```
                         ┌─────────────────────────┐
                         │   TX Node (ESP32 #1)     │
                         │  Soil Moisture | Rain     │
                         │  DHT11 (Temp + Humidity)  │
                         └───────────┬───────────────┘
                                     │
                    ┌────────────────┼────────────────┐
                    │                                  │
              📡 LoRa 433MHz                     📶 WiFi (UDP:4210)
                    │                                  │
                    ▼                                  ▼
       ┌─────────────────────────┐        ┌──────────────────────────┐
       │   RX Node (ESP32 #2)    │        │     Flask ML Server       │
       │  • 16x2 LCD Display      │        │  • RandomForest model     │
       │  • Buzzer alarm          │        │    predicts Safe/Danger   │
       │  • Blynk cloud dashboard │        │  • Logs to MySQL          │
       │  • GSM SMS alert         │        │  • Sends result back to   │
       │    on danger detection   │        │    ESP32 over UDP         │
       └─────────────────────────┘        │  • Live web dashboard     │
                                            │    (auto-refresh, 3s)     │
                                            └──────────────────────────┘
```

**Why decentralized?** Each node can independently trigger a local physical alarm (buzzer/LCD/SMS) via LoRa, without depending on internet connectivity — while the WiFi/UDP path simultaneously feeds the centralized ML pipeline for logging, prediction, and remote dashboard monitoring.

---

## ✨ Features

- 🛰️ **Dual-channel transmission** — LoRa for resilient local alerting, WiFi/UDP for cloud-side ML inference
- 🤖 **ML-based risk classification** — RandomForestClassifier trained on soil/rain/temperature/humidity features
- 🗄️ **Persistent logging** — every reading + prediction stored in MySQL (`esp32_logs`)
- 📺 **Live dashboard** — auto-refreshing web UI showing the latest sensor reading and risk status
- 🔔 **Multi-modal field alerts** — LCD display, buzzer, Blynk IoT app notification, and GSM SMS to registered numbers when danger is detected
- 🧪 **Manual prediction tester** (`check.py`) — a standalone form-based UI to test the model with custom sensor values
- 📊 **Synthetic dataset generator** (`data.py`) — for bootstrapping/training the model without needing months of real field data

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| Microcontroller | ESP32 (x2 — TX & RX nodes) |
| Sensors | Soil moisture sensor, Rain sensor, DHT11 (Temp/Humidity) |
| Long-range comms | LoRa SX1278 (433 MHz) |
| Local alerting | 16x2 LCD, Buzzer, GSM module (SIM800L-class, via AT commands) |
| Cloud IoT dashboard | Blynk |
| Backend server | Python, Flask |
| Machine Learning | scikit-learn (RandomForestClassifier), joblib, NumPy, pandas |
| Database | MySQL |
| Networking | UDP sockets (ESP32 ↔ Flask server) |
| Frontend | HTML/CSS (Jinja2 templates) |

---

## 📁 Repository Structure

```
.
├── Arduino/
│   ├── tx_code/
│   │   └── tx_code.ino        # ESP32 sensor node: reads sensors, sends via LoRa + UDP
│   └── rx_code/
│       └── rx_code.ino        # ESP32 alert node: LoRa receiver, LCD/buzzer/Blynk/GSM alerts
├── templates/
│   └── index.html             # Live web dashboard (auto-refresh every 3s)
├── data.py                    # Generates synthetic training dataset (landslide_data.csv)
├── train.py                   # Trains RandomForestClassifier, saves landslide_model.pkl
├── app.py                     # Flask server + UDP listener + ML inference + MySQL logging
├── check.py                   # Standalone manual prediction tester (web form)
└── README.md
```

---

## ⚙️ Setup & Installation

### 1. Hardware Requirements
- 2x ESP32 dev boards
- 2x LoRa SX1278 modules (433 MHz)
- Soil moisture sensor + Rain sensor (analog)
- DHT11 temperature/humidity sensor
- 16x2 LCD (parallel interface)
- Buzzer
- GSM module (SIM800L or similar, AT-command based)
- Jumper wires, breadboard/PCB, power supply

### 2. Arduino Firmware
Install the following libraries via Arduino IDE Library Manager:
- `WiFi`, `WiFiUdp` (bundled with ESP32 board package)
- `LoRa` (by Sandeep Mistry)
- `DHT sensor library` (Adafruit)
- `LiquidCrystal`
- `BlynkSimpleEsp32`

**Before flashing**, update the following in each `.ino` file with your own credentials (do not commit real values to GitHub):
- `ssid` / `password` — your WiFi network
- `targetIP` in `tx_code.ino` — the local IP address of the machine running `app.py`
- `BLYNK_TEMPLATE_ID` / `BLYNK_TEMPLATE_NAME` / `BLYNK_AUTH_TOKEN` — from your own Blynk console
- GSM destination phone numbers in `rx_code.ino`

Flash `tx_code.ino` to the sensor (TX) ESP32 and `rx_code.ino` to the alert (RX) ESP32.

### 3. Python / Server Setup
```bash
# Clone the repo
git clone https://github.com/mageshit24/A-Decentralized-IoT-and-Machine-Learning-Framework-for-Hyperlocal-Cloudburst-Prediction.git
cd A-Decentralized-IoT-and-Machine-Learning-Framework-for-Hyperlocal-Cloudburst-Prediction

# Install dependencies
pip install flask mysql-connector-python joblib numpy pandas scikit-learn
```

### 4. Database Setup
Create the MySQL database and table used by `app.py`:
```sql
CREATE DATABASE newschema;
USE newschema;

CREATE TABLE esp32_logs (
    id INT AUTO_INCREMENT PRIMARY KEY,
    v1 INT,
    v2 INT,
    v3 INT,
    v4 INT,
    prediction INT,
    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```
Update the `db_config` dictionary in `app.py` with your own MySQL credentials — **ideally loaded from environment variables rather than hardcoded.**

### 5. Train the Model
```bash
python data.py     # generates landslide_data.csv (synthetic data)
python train.py    # trains RandomForestClassifier, saves landslide_model.pkl
```

### 6. Run the Server
```bash
python app.py
```
This starts:
- A UDP listener on port `4210` (receives sensor packets from the ESP32 TX node)
- A Flask web server on `http://0.0.0.0:5000` (live dashboard)

### 7. (Optional) Manual Prediction Tester
```bash
python check.py
```
Opens a simple web form at `http://localhost:5000` to manually enter sensor values and test the model's prediction without hardware.

---

## 🧠 Machine Learning Model

- **Algorithm:** Random Forest Classifier (100 estimators)
- **Features:** `v1` (soil), `v2` (rainfall), `v3` (temperature), `v4` (humidity)
- **Target:** Binary risk label — `0 = Safe`, `1 = Danger`
- **Training/test split:** 80/20
- The current dataset (`data.py`) is **synthetically generated** using threshold-based rules, intended as a placeholder/bootstrap dataset. For production-grade accuracy, retrain on real historical rainfall/landslide data for your target region.

---

## 📡 Data Flow Summary

1. TX ESP32 reads soil, rain, temperature, and humidity sensors.
2. Values are sent via **LoRa** to the RX ESP32 *and* via **WiFi/UDP** to the Flask server.
3. Flask server runs the ML model on the incoming reading and classifies it as Safe (`S`) or Danger (`D`).
4. The result is logged to MySQL and sent back to the TX node over UDP.
5. RX ESP32 (which received the same data via LoRa) displays the reading on its LCD, and if a Danger signal (`D`) was relayed/detected, triggers the buzzer, updates the Blynk dashboard, and sends an SMS alert via the GSM module.
6. The Flask dashboard (`templates/index.html`) auto-refreshes every 3 seconds to show the latest reading and risk status.

---

## 🔒 Security Notes

This is an academic prototype. Before deploying or sharing further, replace all hardcoded credentials (WiFi password, MySQL password, Blynk auth token, GSM phone numbers) with environment variables / a `.gitignore`-excluded config file, and avoid committing secrets to version control.

---

## 🚀 Future Improvements

- Replace synthetic training data with real historical meteorological/landslide datasets
- Add HTTPS and authentication to the Flask dashboard
- Move from polling/UDP to MQTT for more robust IoT messaging
- Add a historical trends chart (not just the latest reading) to the dashboard
- Support multiple sensor nodes per region for true hyperlocal coverage
- Containerize the backend (Docker) for easier deployment

---

## 👤 Author

**Magesh**
Final Year Major Project — IoT & Machine Learning for Disaster Risk Prediction

---

## 📄 License

This project is currently unlicensed. Consider adding an open-source license (e.g., MIT) if you'd like others to reuse or contribute to it.
