# ESP8266 Water Pump Controller

## Component Repositories

- Android App 
 https://github.com/xlcMitchell/WaterPumpApp
  

- ESP8266 Firmware (MQTT Subscriber & Pump Control)  
  https://github.com/xlcMitchell/WaterPumpServer1.0



## Overview
This repository contains the **ESP8266 firmware** for a smart plant watering system.  
The ESP8266 connects to Wi-Fi, subscribes to MQTT commands via **HiveMQ Cloud (TLS)**, and controls a water pump through a relay.

The pump is activated remotely by publishing an MQTT command and is automatically switched off after a fixed duration as a safety measure.

This firmware is part of a larger **end-to-end IoT system** that includes an Android app and system-level documentation.

---

## System Behaviour
- ESP8266 connects to Wi-Fi on boot
- Securely connects to HiveMQ Cloud using MQTT over TLS (port 8883)
- Subscribes to pump command topic
- Activates pump for a predefined duration

---

## Features
- MQTT-based remote pump control
- Secure TLS connection to HiveMQ Cloud
- Automatic pump shut-off (safety timeout)
- Credentials isolated in `config.h` (excluded from Git)

---

## MQTT Topics
| Topic | Direction | Description |
|------|----------|------------|
| `plant/pump/on` | App > ESP8266 | Pump control command (`on`) |


---

## Hardware Overview
- ESP8266 Dev Board
- Relay module (pump switching)
- External DC power supply
- Linear regulator (LM7805) for onboard 5V regulation
- Common ground shared between ESP8266 and pump control circuitry

## Demo
[![Pump Demo](images/demo.png)](https://youtu.be/YourVideoID)

- `demo.png` is a screenshot of the pump or app in action.
- Clicking the image opens the hosted demo video 

![PCB 3D View](images/demo1.png)

![PCB View](images/demo2.png)

![PCB With ESP8266](images/demo4.png)

![PCB Without ESP8266](images/demo5.png)

