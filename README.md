# WM--Abhishek-mp-smartcity
# Smart Waste Bin Monitoring & Collection Optimization System  
**Virtual IoT Design Challenge – Smart City Living Lab (IIITH)**

##  Project Overview
Urban waste management systems often suffer from inefficient collection schedules, overflowing bins, and unnecessary operational costs. This project proposes a **Smart Waste Bin Monitoring and Collection Optimization System** using Internet of Things (IoT) concepts to enable real-time monitoring and intelligent decision-making for waste collection.

The system monitors waste bin fill levels, notifies authorities when bins are near capacity, and generates optimized routes for garbage collection vehicles.

---

## Objectives
- Monitor fill levels of waste bins across different city zones
- Automatically notify authorities when bins exceed a threshold level
- Optimize garbage truck routes to reduce fuel consumption and time
- Design a scalable, low-power, and cost-effective IoT architecture

---

##  System Architecture

### Hardware Components
- **Ultrasonic Sensor:** Measures distance to waste surface to estimate fill level
- **ESP32 Microcontroller:** Processes sensor data and manages communication
- **Communication Module:** LoRaWAN / Wi-Fi depending on deployment area
- **Power Supply:** Battery with optional solar panel
- **Optional Sensors:** Temperature or tilt sensor for safety and tamper detection

### Software Components
- Embedded firmware on ESP32
- Cloud-based data processing and storage
- Web dashboard for visualization and alerts

---

##  Data Flow Design
1. Ultrasonic sensor measures the distance to waste
2. ESP32 converts distance into fill percentage
3. Data is transmitted via LoRaWAN or Wi-Fi
4. Cloud server receives and stores the data
5. Alerts and route optimization logic are executed
6. Dashboard displays real-time bin status

---

##  Communication Protocols
- **MQTT:** Lightweight and efficient for IoT data transmission
- **HTTP/REST:** Used for dashboard and reporting services

**Reason for Selection:**  
MQTT minimizes bandwidth usage and power consumption, making it suitable for large-scale IoT deployments.

---

##  Route Optimization Strategy

### Decision Logic
- Bins with fill level **greater than 80%** are marked for collection
- Bins are grouped based on geographic proximity
- Shortest path algorithms are used to generate optimized routes

### Sample Pseudocode

If bin_fill > 80%:
Add bin to priority list
Group bins by location
Compute shortest collection route
Assign route to garbage truck


---

## Power Management Plan
- Periodic sensing instead of continuous operation
- ESP32 operates in deep sleep mode
- Low-power LoRa communication
- Optional solar charging to extend battery life

**Estimated battery life:** 6–12 months

---

##  Reliability & Fault Handling
- Multiple sensor readings averaged to reduce noise
- Automatic calibration when bin is empty
- Detection of abnormal or stagnant sensor data
- Manual override option via dashboard

---

## Scalability & Network Considerations
- Supports deployment of **100+ bins across multiple zones**
- **Star topology** (Bin → Gateway → Cloud) used for simplicity and efficiency
- Easy expansion by adding additional gateways

---

##  Cost & Feasibility (Approximate per Bin)

| Component | Estimated Cost (INR) |
|--------|---------------------|
| Ultrasonic Sensor | 150 |
| ESP32 + LoRa Module | 600 |
| Battery & Power Circuit | 300 |
| Enclosure & Mounting | 250 |
| **Total** | **1300 – 1500** |

### Trade-Offs
- Higher accuracy sensors increase cost
- LoRa provides better scalability than Wi-Fi
- Cloud analytics add operational cost but improve intelligence

---

##  Repository Structure

---

## Key Highlights
- End-to-end IoT system design
- Low-power and scalable architecture
- Practical smart city use case
- Conceptual design suitable for real-world deployment

---
Hardware/
 └── Sensor_Code/
     └── esp32_ultrasonic_mqtt.ino
     #include <WiFi.h>
#include <PubSubClient.h>

// -------- WiFi Credentials --------
const char* ssid = "YOUR_WIFI_NAME";
const char* password = "YOUR_WIFI_PASSWORD";

// -------- MQTT Broker --------
const char* mqttServer = "broker.hivemq.com";
const int mqttPort = 1883;
const char* mqttTopic = "smartcity/wastebin/filllevel";

// -------- Ultrasonic Pins --------
#define TRIG_PIN 5
#define ECHO_PIN 18

WiFiClient espClient;
PubSubClient client(espClient);

// -------- Bin Parameters --------
const float BIN_HEIGHT_CM = 50.0;   // Height of bin

void setup() {
  Serial.begin(115200);

  pinMode(TRIG_PIN, OUTPUT);
  pinMode(ECHO_PIN, INPUT);

  connectWiFi();
  client.setServer(mqttServer, mqttPort);
}

void loop() {
  if (!client.connected()) {
    connectMQTT();
  }
  client.loop();

  float distance = getDistance();
  float fillLevel = calculateFillLevel(distance);

  publishData(fillLevel);

  // Deep sleep for power saving (30 minutes)
  esp_sleep_enable_timer_wakeup(30 * 60 * 1000000ULL);
  esp_deep_sleep_start();
}

// -------- Functions --------

void connectWiFi() {
  WiFi.begin(ssid, password);
  Serial.print("Connecting to WiFi");
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("\nWiFi connected");
}

void connectMQTT() {
  while (!client.connected()) {
    Serial.print("Connecting to MQTT...");
    if (client.connect("ESP32_WasteBin")) {
      Serial.println("connected");
    } else {
      delay(2000);
    }
  }
}

float getDistance() {
  digitalWrite(TRIG_PIN, LOW);
  delayMicroseconds(2);
  digitalWrite(TRIG_PIN, HIGH);
  delayMicroseconds(10);
  digitalWrite(TRIG_PIN, LOW);

  long duration = pulseIn(ECHO_PIN, HIGH, 30000);
  float distance = duration * 0.034 / 2;
  return distance;
}

float calculateFillLevel(float distance) {
  if (distance > BIN_HEIGHT_CM) distance = BIN_HEIGHT_CM;
  float fill = ((BIN_HEIGHT_CM - distance) / BIN_HEIGHT_CM) * 100;
  return fill;
}

void publishData(float fillLevel) {
  char payload[50];
  snprintf(payload, sizeof(payload), "Fill Level: %.2f %%", fillLevel);

  client.publish(mqttTopic, payload);
  Serial.println(payload);
}

## 🧪 Sample Implementation Code
This repository includes a sample ESP32 firmware that:
- Reads ultrasonic sensor data
- Calculates waste bin fill percentage
- Publishes data using MQTT protocol
- Uses deep sleep for power efficiency

The code is provided for conceptual demonstration and can be extended for LoRaWAN or NB-IoT deployments.


## Author
**Abhishek**  
Virtual IoT Design Challenge  
Smart City Living Lab – IIITH
