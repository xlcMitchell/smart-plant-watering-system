## Wi-Fi & MQTT Connection Test
- [x] Upload the firmware to the ESP8266.
- [x] Verify Wi-Fi connection on the Serial Monitor.
- [x] Verify MQTT connection and subscription to `plant/#`.

### Communication and Network Testing

| Test ID | Test Description | Test Steps | Expected Result | Actual Result | Status |
|-------|------------------|-----------|-----------------|---------------|--------|
| T01 | ESP connects to Wi-Fi (LAN) | Power ESP on within home Wi-Fi network | ESP connects and obtains IP address | Connected successfully, IP assigned | [x] |
| T02 | MQTT broker connection | ESP attempts to connect to MQTT broker | MQTT connection established | Connected to HiveMQ broker | [x] |
| T03 | MQTT topic subscription | ESP subscribes to `plant/pump/on` topic | Subscription successful | Subscribed successfully | [x] |
| T04 | MQTT message receive (LAN) | Publish `on` to `plant/pump/on` from same network | ESP receives message | Message received correctly | [x] |
| T05 | MQTT message receive (Mobile Data) | Publish `on` to `plant/pump/on` via mobile network | ESP receives message | Message received correctly | [x] |
| T06 | Cross-network MQTT messaging | ESP on Wi-Fi, client on mobile data | Message received without delay | Successful cross-network delivery | [x] |

### Notes
- Relay hardware was not connected during this test phase.
- Testing focused on Wi-Fi connectivity and MQTT messaging.
- Relay and pump activation tests will be added in a later phase.

# Post-Soldering Test Checklist

## Visual Inspection
- [x] Check all solder joints for cold solder points or bridges.
- [x] Confirm correct orientation of all components (e.g., polarized components like capacitors or diodes).
- [x] Ensure the ESP8266 is seated correctly with proper pin alignment.


# ESP8266 Power Debugging – Test Summary

## System Under Test
- **MCU:** ESP8266 (dev board)
- **Power sources tested:**  
  - 9V battery  
  - External power supply (2A rated)  
  - USB power
- **Power path:**  
  External supply → 5V linear regulator → ESP8266 VIN  
  ESP8266 onboard 3.3V regulation

---

## Test Summary Table

| Test # | Test Description | Setup / Action | Observation | Conclusion |
|------|------------------|----------------|-------------|------------|
| 1 | Measure 9V battery (unloaded) | Battery disconnected from circuit | ~9V measured | Battery OK unloaded |
| 2 | Measure 9V battery under load | Battery connected with ESP installed | Voltage dropped to ~3V | Battery cannot sustain load |
| 3 | Regulator temperature check | 9V → 5V linear regulator active | Regulator became hot | Regulator overloaded |
| 4 | Measure ESP VIN voltage | ESP connected, powered via VIN | ~1.6V at VIN | Upstream voltage collapsing |
| 5 | Remove ESP8266 | ESP removed from circuit | 9V at regulator input, 5V output stable | ESP rail causing collapse |
| 6 | Replace battery with 2A PSU | 2A-rated supply → regulator | Correct input voltage, regulator still hot | PSU OK, regulator limiting |
| 7 | Measure VIN with 2A PSU | ESP connected | VIN still ~1.6V | Regulator in current/thermal limit |
| 8 | Power ESP via USB only | VIN disconnected, USB powered | ESP stable | Confirms external power path faulty |
| 9 | Resistance test 3.3V → GND (ESP plugged in) | Multimeter, power off | 3.3V rail shorted to GND | Hard short detected |
| 10 | Resistance test with ESP removed | ESP removed | No short on 3.3V rail | Short introduced via ESP/header |
| 11 | Header pin continuity check | Checked ESP header pins | Suspected 3.3V pin connected to GND | Pin misidentified |
| 12 | Root cause identified | Compared pinout vs wiring | Wrong ESP header pin tied to ground | Wiring / pin-mapping error |

---

## Root Cause
- Incorrect ESP8266 header pin was connected to ground.
- The pin believed to be **3.3V** was actually a **GND pin**.
- This caused the 3.3V rail to be shorted, leading to regulator overheating and voltage collapse.

---

## Corrective Actions
- Correct ESP8266 header pin mapping
- Remove incorrect ground connection
- Verify pinouts using continuity before power-up
- Power ESP via USB for initial validation


## GPIO & Relay Test
- [ ] Test the GPIO pin controlling the relay (no pump connected).
- [ ] Confirm the relay clicks on and off when receiving `on` command.

## Pump Connection Test
- [ ] Connect the pump to the relay output.
- [ ] Send an `on` command and confirm the pump runs for the correct duration and then stops.

## Final System Check
- [ ] Run a full cycle using the Android app to trigger the watering process.
- [ ] Observe for any heat issues or instability during operation.
