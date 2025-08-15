# 🕒 Smart Watch Real-Time Streaming Platform (AWS PoC)

A proof-of-concept **real-time health monitoring system** for smart watches using AWS IoT Core, Kinesis, Lambda, DynamoDB, and SNS.
It simulates live biometric data (heart rate, steps, SpO₂) from wearable devices, processes it in real time, stores it, and triggers alerts if health thresholds are exceeded.

---

## 📂 Project Structure

```
smartwatch-poc/
│
├── template.yaml               # AWS SAM template (deployable architecture)
├── README.md                   # Project documentation
│
├── src/
│   └── lambda_function.py      # Lambda for processing and alerting
│
└── simulator/
    └── smartwatch_simulator.py # Python MQTT publisher simulating watch data
```

---

## 🛠 Architecture Overview

**Data Flow**:

1. Smart watch (or simulator) publishes biometric data → **AWS IoT Core (MQTT)**.
2. **IoT Rule** forwards data to **Amazon Kinesis Data Stream**.
3. **AWS Lambda** processes data:

   * Stores in **DynamoDB** (time-series storage).
   * Sends **SNS alerts** if thresholds exceeded (e.g., heart rate > 150 bpm).
4. Data available for **real-time dashboards** or analytics.

![AWS Architecture](https://user-images.githubusercontent.com/000000/0000000/aws-smartwatch-arch.png) <!-- optional diagram -->

---

## 📋 Requirements

* [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html) configured
* [AWS SAM CLI](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/install-sam-cli.html)
* Python 3.9+
* MQTT certificates for AWS IoT Core (for simulator)
* IAM permissions to deploy SAM stacks

---

## 🚀 Deployment

### 1️⃣ Clone & Build

```bash
git clone https://github.com/yourusername/smartwatch-poc.git
cd smartwatch-poc

sam build
```

---

### 2️⃣ Deploy Stack

```bash
sam deploy --guided
```

* **Stack Name**: `smartwatch-poc`
* **Region**: your preferred AWS region
* Accept all defaults unless you need custom values.

---

## 📡 Run the Simulator

### Install dependencies:

```bash
pip install paho-mqtt
```

### Example simulator (`simulator/smartwatch_simulator.py`):

```python
import json, time, random
import paho.mqtt.client as mqtt

BROKER_ENDPOINT = "YOUR_IOT_ENDPOINT"  # from AWS IoT Core
PORT = 8883
TOPIC = "smartwatch/data"

CA_PATH = "AmazonRootCA1.pem"
CERT_PATH = "device-certificate.pem.crt"
KEY_PATH = "private.pem.key"

client = mqtt.Client()
client.tls_set(ca_certs=CA_PATH, certfile=CERT_PATH, keyfile=KEY_PATH)

client.connect(BROKER_ENDPOINT, PORT, keepalive=60)

while True:
    payload = {
        "deviceId": "watch-001",
        "timestamp": int(time.time()),
        "heartRate": random.randint(60, 180),
        "steps": random.randint(0, 100),
        "spo2": random.randint(90, 100)
    }
    client.publish(TOPIC, json.dumps(payload), qos=1)
    print("Sent:", payload)
    time.sleep(5)
```

Run:

```bash
python simulator/smartwatch_simulator.py
```

---

## 🔍 Testing

* **DynamoDB** → Table `SmartWatchData` should receive records.
* **Kinesis** → View incoming records from IoT Core.
* **SNS** → Subscribe an email/phone and receive alerts when heart rate > 150 bpm.

---

## 📦 Cleanup

To remove all AWS resources:

```bash
sam delete
```

---

## 📌 Notes

* You can easily extend this to add **API Gateway WebSocket** + a **live dashboard**.
* The PoC uses **PAY\_PER\_REQUEST** DynamoDB and **1 shard** in Kinesis for simplicity; scale based on device count.
* IoT Core requires **device provisioning** with valid X.509 certificates.

---

## 📜 License

MIT License – feel free to modify and use.

---

If you want, I can **add the IoT Core certificate creation commands** and **WebSocket dashboard extension** to the README so it becomes a complete end-to-end developer guide.

Do you want me to expand it with that next?
