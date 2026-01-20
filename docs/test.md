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

# Testing – Watering History Feature

This test list verifies the **watering history**, **last-watered tracking**, and **persistence** features added to the Android app.

Tests are marked complete only when behaviour is verified in the running system.

---

## History Update Tests

- [x] **H1 – History entry added on DONE**
  - Trigger watering until pump status becomes `DONE`
  - Verify a new timestamp entry appears at the top of the history list

- [x] **H2 – Last watered text updates**
  - Trigger watering until `DONE`
  - Verify `txtLastRun` displays the latest watering timestamp

- [x] **H3 – History order (newest first)**
  - Trigger multiple waterings
  - Verify newest entry appears at the top of the list

- [x] **H4 – History size limit enforced**
  - Trigger watering more than `HISTORY_MAX` times
  - Verify history list does not exceed the maximum size
  - Verify oldest entries are removed first

---

## Persistence Tests

- [x] **H5 – History persists after app restart**
  - Add 2–3 history entries
  - Close the app completely
  - Reopen the app
  - Verify history list is restored correctly

- [x] **H6 – Last watered persists after app restart**
  - Add a history entry
  - Restart the app
  - Verify `txtLastRun` still displays the latest timestamp

  History working after editing code to save shared preference for history setter method

---

## Edge Case & Reliability Tests

- [x] **H7 – No history entry without DONE**
  - Send watering command
  - Force ESP offline before `DONE`
  - Verify no new history entry is added

- [x] **H8 – No duplicate entries**
  - Trigger a single watering cycle
  - Verify only one history entry is added for that cycle

- [x] **H9 – UI thread safety**
  - Observe app during incoming MQTT messages
  - Verify no crashes or UI exceptions occur

---

## Regression Tests

- [x] **H10 – Pump control still works**
  - Verify Water Plant button still sends command correctly
  - Verify pump activates and stops as expected

- [x] **H11 – Online/Offline indicator unaffected**
  - Toggle ESP online/offline
  - Verify status dot and text still update correctly

---

## Definition of Done

This feature is considered complete when:

- [x] History updates only on `DONE`
- [x] History is capped to the configured maximum size
- [x] History and last-watered time persist across app restarts
- [x] UI updates are stable and crash-free

## Auto-Watering + History – Test Log (Current Findings)

### Observations / Notes
- [x] History sometimes did not show latest entry at top after app restart
- [x] Reproduced: watered twice, closed app, only one entry retained
- [x] Root cause found: `apply()`/`commit()` missing in SharedPreferences history setter
- [x] Auto-watering tested and working
- [x] Auto-watering triggers after next moisture reading interval (expected)
- [x] Verified no immediate re-water on next 6-minute reading (cooldown/latch behaving)
- [x] Moisture rose from ~0% (dry) to ~34% after one watering

---

## Suggested Test Checklist (Next Steps)

### A) History Persistence (App)
- [x] **Restart ordering check**: Water 3 times (with clear timestamps), force close app, reopen → newest entry appears at top
- [x] **Cold start after phone restart**: Water once, reboot phone, open app → history + last watered still correct
- [x] **Rapid close test**: Trigger pump, wait for `DONE`, immediately swipe app away → reopen → history entry present
- [x] **Max history trim**: Water > `HISTORY_MAX` times → verify list trims oldest entries and still saves correctly
- [x] **Duplicate prevention**: Ensure only one history entry is added per pump cycle (`RUNNING`→`DONE`) even if `DONE` arrives twice


### b) Auto-Watering Logic
- [x] **Auto OFF**: Set `enabled=0`, set dry threshold high → verify it never waters automatically
- [x] **Threshold boundary test**: Set threshold = current moisture ±1% → verify correct behaviour around edge
- [x] **Cooldown enforcement**: Force an auto-water event, then ensure no further watering until cooldown 
- [ ] **Manual + Auto interaction**: Manual water while auto enabled → ensure it doesn’t immediately auto-trigger again on next reading

### c) Sensor + Mapping Validation
- [x] **Raw vs % sanity**: Log RAW + % before and after watering → % should increase after watering
- [ ] **Repeatability**: Take 5 readings in the same condition → confirm noise is within a small band (consider averaging if noisy)
- [ ] **Post-water stabilization**: Compare moisture immediately after watering vs 10 minutes later (expect changes as water spreads)

### d) Failure / Recovery Tests
- [ ] **WiFi drop recovery**: Turn router off briefly → ESP reconnects and resumes normal operation without watering unexpectedly
- [ ] **MQTT reconnect**: Restart broker session / force disconnect → device reconnects, resubscribes, and receives retained config
- [ ] **ESP reboot safety**: Reboot ESP while soil is dry → verify it does NOT water until after it receives config + next scheduled reading (or confirm expected behaviour)

---

## GPIO & Relay Test
- [ ] Test the GPIO pin controlling the relay (no pump connected).
- [ ] Confirm the relay clicks on and off when receiving `on` command.

## Pump Connection Test
- [ ] Connect the pump to the relay output.
- [ ] Send an `on` command and confirm the pump runs for the correct duration and then stops.

## Final System Check
- [ ] Run a full cycle using the Android app to trigger the watering process.
- [ ] Observe for any heat issues or instability during operation.
