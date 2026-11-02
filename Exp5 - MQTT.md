## Steps

### Step 1: Connect the DHT11 sensor

Connect the DHT11 sensor to Raspberry Pi 5 as follows:

| DHT11 pin | Raspberry Pi 5 |
|---|---|
| VCC | 3.3V (physical pin 1) |
| DATA | GPIO4 (physical pin 7) |
| GND | GND (physical pin 6) |

Use the appropriate wiring for your three-pin or four-pin DHT11 module.

### Step 2: Create a Python virtual environment

```bash
python3 -m venv ~/mqtt-env
source ~/mqtt-env/bin/activate
```

The terminal prompt should show `(mqtt-env)`. If the environment already exists, activate it instead of creating it again.

### Step 3: Install the required libraries

```bash
pip install adafruit-blinka
pip install adafruit-circuitpython-dht paho-mqtt
```

For the `lgpio` dependency on Raspberry Pi OS:

```bash
sudo apt update
sudo apt install python3-lgpio -y
```

### Step 4: Create the publisher program

```bash
nano mqtt_dht11_rpi5.py
```
Or create and save using thonny.

### Step 5: Run the publisher

```bash
python mqtt_dht11_rpi5.py
```

### Step 6: Open a second terminal for the subscriber

Open another terminal window. Install the MQTT command-line clients:

```bash
sudo apt-get update
sudo apt-get install mosquitto-clients -y
```

Subscribe to the topic:

```bash
mosquitto_sub -h broker.hivemq.com -t "rpi5/dht11/data"
```

The subscriber should display the JSON messages published by the first terminal.


### Step 7: Create the Python subscriber **(optional)**

Activate the environment in the second terminal:

```bash
source ~/mqtt-env/bin/activate
```

Create the subscriber file:

```bash
nano mqtt_subscriber.py
```

Paste the Python subscriber code, save it, and run:

```bash
python mqtt_subscriber.py
```

## Code

### Publisher — `mqtt_dht11_rpi5.py`

```python
import time
import board
import adafruit_dht
import paho.mqtt.client as mqtt
import json

# MQTT broker details
MQTT_BROKER = "broker.hivemq.com"
MQTT_PORT = 1883
MQTT_TOPIC = "rpi5/dht11/data"

# Initialize DHT11 sensor on GPIO4 (Pin 7)
dht_sensor = adafruit_dht.DHT11(board.D4)

# Create MQTT client
client = mqtt.Client(client_id="rpi5-dht11")
client.connect(MQTT_BROKER, MQTT_PORT, 60)

print("Sending DHT11 data to MQTT broker...")

try:
    while True:
        try:
            temperature = dht_sensor.temperature
            humidity = dht_sensor.humidity

            if temperature is not None and humidity is not None:
                data = {
                    "temperature": temperature,
                    "humidity": humidity
                }
                payload = json.dumps(data)
                print("Sending:", payload)
                client.publish(MQTT_TOPIC, payload)
            else:
                print("Sensor returned None")

        except RuntimeError as e:
            print("Sensor read error:", e)

        time.sleep(5)

except KeyboardInterrupt:
    print("Stopped by user.")

finally:
    client.disconnect()
    print("MQTT disconnected.")
```

### Subscriber **(Optional)** — `mqtt_subscriber.py`

```python
import paho.mqtt.client as mqtt
import json

# Called when client connects to broker
def on_connect(client, userdata, flags, rc):
    print("Connected with result code", rc)
    client.subscribe("rpi5/dht11/data")

# Called when message is received
def on_message(client, userdata, msg):
    try:
        data = json.loads(msg.payload.decode())
        temperature = data.get("temperature")
        humidity = data.get("humidity")
        print(f"Temp: {temperature}°C | Humidity: {humidity}%")
    except Exception as e:
        print("Error decoding message:", e)

# Set up MQTT client
client = mqtt.Client()
client.on_connect = on_connect
client.on_message = on_message

# Connect to public broker
client.connect("broker.hivemq.com", 1883, 60)

# Keep listening forever
client.loop_forever()
```
