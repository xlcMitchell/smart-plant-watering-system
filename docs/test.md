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

# Power & Stability Testing - Root Cause Investigation

This section documents the testing performed to diagnose instability in the ESP8266 water pump system, including cases where the pump only operated once or twice, failed to turn off correctly, or caused the ESP8266 to reset.

---

## Test Environment
- ESP8266 dev board
- Relay-controlled DC water pump
- MQTT control via HiveMQ
- Linear regulator: LM7805
- Common ground between logic and pump
- Multiple power sources tested

---

## Test Group A - 9V PP3 Battery

### Tests
- [x] Measure battery voltage with no load
- [x] Measure battery voltage under system load
- [x] Trigger pump via MQTT
- [x] Observe ESP8266 LED behaviour
- [x] Observe relay behaviour
- [x] Observe pump run duration

### Observations
- Battery measured ~9V unloaded
- Voltage collapsed significantly under load
- ESP8266 reset observed (LED flash)
- Relay produced buzzing / chattering noise
- Pump ran longer than intended or unpredictably

### Conclusion
Rectangular 9V PP3 battery cannot supply sufficient current. High internal resistance causes voltage sag, brownouts, ESP resets, and relay instability.

---

## Test Group B - 12V DC Supply + LM7805

### Tests
- [x] Power system using 12V DC adapter
- [x] Trigger pump multiple times via MQTT
- [x] Monitor regulator temperature
- [x] Observe ESP reset behaviour

### Observations
- System worked once or twice, then became unstable
- LM7805 became excessively hot
- ESP8266 reset after pump activation
- Required power cycle to recover

### Conclusion
12V input causes excessive power dissipation in the LM7805. Thermal stress and voltage sag lead to brownouts and unstable operation.

---


## Root Cause Summary
- ESP8266 requires high transient current for Wi-Fi transmission
- Inadequate current delivery causes voltage sag and brownouts
- Linear regulation from high input voltage leads to overheating
- Power instability results in resets, relay chatter, and timing failures

---

## Final Conclusion
System instability was caused by **power delivery limitations**, not firmware logic. Reliable operation requires:
- DC power supply rated > 500mA (1A preferred)
- Reduced regulator dissipation (lower input voltage or buck converter)
- Proper bulk decoupling near ESP8266 VIN

---

## Test Group 1: Android App - MQTT Message Receiving

| Test ID | Test Description | Steps Performed | Expected Result | Actual Result | Status | Notes |
|------|------------------|-----------------|-----------------|---------------|--------|------|
| APP-01 | Receive pump status message | Publish `plant/pump/status = RUNNING` from ESP / HiveMQ | App receives message and updates pump status text | Pump status received and displayed correctly |  Pass | Status text updates as expected |
| APP-02 | Receive pump idle/done message | Publish `plant/pump/status = DONE` | App updates pump status text to DONE | Status updated correctly |  Pass | No UI issues observed |
| APP-03 | Receive device OFFLINE message (LWT) | Disconnect ESP from broker | App receives OFFLINE message | OFFLINE received but delayed |  Partial | Delay likely due to broker/session timing |
| APP-04 | Receive device ONLINE message | ESP connects to broker | App receives ONLINE message | ONLINE received successfully |  Pass | Retained message works |
| APP-05 | UI updates on MQTT message | Observe TextView updates on message arrival | Text fields update correctly | Text fields updated correctly |  Pass | UI thread handling confirmed |
| APP-06 | ESP auto-reconnect visibility | ESP goes offline due to power issue | App shows ESP returning ONLINE | ESP does not reconnect |  Fail | Root cause identified as power instability |

---

## Test Group 2: Power & Hardware Stability - Water Pump Operation

| Test ID | Test Description | Setup | Expected Result | Actual Result | Status | Notes |
|------|------------------|-------|-----------------|---------------|--------|------|
| PWR-01 | Pump activation under VIN power | ESP + pump powered from VIN rail | Pump runs, ESP remains online | ESP goes offline after 1-3 activations |  Fail | Indicates voltage sag/brownout |
| PWR-02 | Repeated pump cycles | Trigger pump multiple times | Stable operation | ESP disconnects | Fail | Failure reproducible |
| PWR-03 | Add bulk + decoupling capacitors | Added 1000uF electrolytic + 0.1uF ceramic across VIN/GND | Improved stability | No improvement |  Fail | Caps insufficient for sustained sag |
| PWR-04 | Flyback diode present | Pump has flyback diode | Reduced noise | Still unstable |  Partial | Noise not primary issue |
| PWR-05 | Separate ESP power (USB) | ESP powered via USB, pump on original supply | Stable operation | Works perfectly |  Pass | Confirms VIN power path as root cause |
| PWR-06 | ESP reconnect behaviour | ESP powered from VIN after failure | ESP reconnects to MQTT | ESP does not reconnect |  Fail | ESP likely browning out/resetting |

---

## Summary of Findings

- Android app MQTT message receiving and UI updates are functioning correctly.
- OFFLINE (LWT) messages are received but may be delayed due to broker timing.
- ESP8266 fails to reconnect after going offline when powered via VIN.
- Adding capacitors alone did not resolve power instability.
- Powering the ESP8266 from a separate, stable 5V USB supply completely resolves the issue.
- Root cause identified as **VIN power rail instability under pump load**.

## Conclusion

The issue is hardware-related (power delivery), not MQTT logic or Android implementation.  
Recommended fix is to power the ESP8266 from a **dedicated regulated rail (buck converter)** while keeping the pump on the main supply, with a shared ground.

## GPIO & Relay Test
- [ ] Test the GPIO pin controlling the relay (no pump connected).
- [ ] Confirm the relay clicks on and off when receiving `on` command.

## Pump Connection Test
- [ ] Connect the pump to the relay output.
- [ ] Send an `on` command and confirm the pump runs for the correct duration and then stops.

## Final System Check
- [ ] Run a full cycle using the Android app to trigger the watering process.
- [ ] Observe for any heat issues or instability during operation.
