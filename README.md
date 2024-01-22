# MQTT-ThingsBoard
- This is a sample code between Raspberry and the DHT22 Sensor
- What are we need:
	- Raspberry Pi 3 (or higher).
	- Sensor DHT22.
	- Account ThingsBoard.
- First, I need install **Raspberry OS** for your PI.
- Setup **SSH** for this raspberry device.
- Follow connect between Pi and sensor:

Raspberry	|	Sensor DHT22	|
----------------|-----------------------|
5VDC		|	5 VDC		|
GND		|	GND		|
GPIO 4		|	Data		|

- ![DHT22_Connect_RPi](https://github.com/VinhLin/MQTT-ThingsBoard/blob/main/Image/DHT22_Connect_RPi.png)

---------------------------------------------------------------------------
## Install software and read data
- I need SSH in your raspberry. And update:
```
sudo apt update
sudo apt-get upgrade
```
### Install some tools
- Install Pip:
```
sudo apt install python3-pip
pip --version
```
- Install Git:
```
sudo apt install git
git --version
```

### Install Adafruit_CircuitPython_DHT
```
pip3 install adafruit-circuitpython-dht
sudo apt-get install libgpiod2
cd
mkdir lab1-mqtt
cd lab1-mqtt
git clone https://github.com/adafruit/Adafruit_CircuitPython_DHT.git
cd Adafruit_CircuitPython_DHT
python3 -m pip install -r requirements.txt
```

### Test Connection:
```
cd example
nano dht_simpletest.py
```
- Change content `dhtDevice = adafruit_dht.DHT22(board.D18)` to `dhtDevice = adafruit_dht.DHT22(board.D4)`
- ![Picture_1](https://github.com/VinhLin/MQTT-ThingsBoard/blob/main/Image/Picture_1.png)
- Run:
```
python3 dht_simpletest.py
```
- ![Result - Picture 2](https://github.com/VinhLin/MQTT-ThingsBoard/blob/main/Image/Picture_2.png)

-------------------------------------------------------------------------------
## ThingsBoard
```
THINGSBOARD_HOST = 'demo.thingsboard.io'
THINGSBOARD_HOST = 'thingsboard.cloud'
```

### Send Data to ThingsBoard
```
cd Adafruit_CircuitPython_DHT
mkdir thingsboard-dht22
```
- Source:
```
import os
import time
import sys
import adafruit_dht
import board
import paho.mqtt.client as mqtt
import json

# Initial the dht device, with data pin connected to:
dhtDevice = adafruit_dht.DHT22(board.D4)

THINGSBOARD_HOST = 'demo.thingsboard.io'
ACCESS_TOKEN = 'XXXXXX'

# Data capture and upload interval in seconds. Less interval will eventually hang the DHT22.
INTERVAL=2

sensor_data = {'temperature': 0, 'humidity': 0}

next_reading = time.time() 

client = mqtt.Client()

# Set access token
client.username_pw_set(ACCESS_TOKEN)

# Connect to ThingsBoard using default MQTT port and 60 seconds keepalive interval
client.connect(THINGSBOARD_HOST, 1883, 60)

client.loop_start()

try:
    while True:

        temperature = dhtDevice.temperature
        humidity = dhtDevice.humidity

        print(
            "Temp: {:.1f} C    Humidity: {}% ".format(
                temperature, humidity
            )
        )
        sensor_data['temperature'] = temperature
        sensor_data['humidity'] = humidity

        # Sending humidity and temperature data to ThingsBoard
        client.publish('v1/devices/me/telemetry', json.dumps(sensor_data), 1)

        next_reading += INTERVAL
        sleep_time = next_reading-time.time()
        if sleep_time > 0:
            time.sleep(sleep_time)
except KeyboardInterrupt:
    pass

client.loop_stop()
client.disconnect()
```





