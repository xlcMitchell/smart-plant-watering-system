# Smart Plant Watering System – Agile Task List

This task list follows an **Agile / iterative approach**. Work is organised into **iterations (sprints)**, each producing a testable outcome. Tasks are only marked complete when verified.

---

## Sprint 0 – Discovery & Foundations
**Goal:** Define scope, tools, and success criteria

- [x] Define project goal (remote plant watering)
- [x] Choose ESP8266 platform
- [x] Choose MQTT for communication
- [x] Choose HiveMQ Cloud as broker
- [x] Decide on Android app as control interface
- [x] Create GitHub repository
- [x] Create initial README and other documentation

**Done when:** Architecture decisions documented

---

## Sprint 1 – Hardware v1 (Initial PCB Design)
**Goal:** Create first working hardware design

- [x] Select ESP8266 dev board
- [x] Design custom PCB in EasyEDA
- [x] Place headers and power circuitry
- [x] Order PCB (v1)
- [x] Discover ESP8266 pin spacing issue (25.1 mm vs 25.4 mm)
- [x] Document PCB design mistake and lesson learned

**Done when:** PCB v1 reviewed and issues identified

---

## Sprint 2 – Hardware v2 (PCB Redesign)
**Goal:** Correct PCB and prepare for reliable assembly

- [x] Re-measure ESP8266 header spacing correctly (25.4 mm)
- [x] Update PCB footprint with correct spacing
- [x] Review hole sizes and tolerances
- [x] Update silkscreen and pin labels
- [x] Order revised PCB (v2)
- [x] Assemble revised PCB
- [x] Solder components
- [x] Power-on test (no load)

**Done when:** PCB v2 powers correctly

---

##  Sprint 3 – MQTT Broker Setup
**Goal:** Establish reliable cloud communication

- [x] Create HiveMQ Cloud cluster
- [x] Configure TLS broker connection
- [x] Understand auto-generated credentials
- [x] Create credentials for Android app
- [x] Rotate credentials after exposure
- [x] Secure credentials using .gitignore
- [x] Verify publish/subscribe using HiveMQ Web Client
- [x] Learn MQTT topic wildcard rules

**Done when:** Messages visible in Web Client

---

##  Sprint 4 – Android App (MQTT Publisher)
**Goal:** Publish MQTT commands from Android

- [x] Create clean Android project
- [x] Evaluate MQTT libraries for Android
- [x] Select Paho MQTT client
- [x] Add minimal dependency only
- [x] Add INTERNET permission
- [x] Implement MQTTHelper class
- [x] Handle async MQTT connection
- [x] Generate unique MQTT client ID per session
- [x] Publish command from button press
- [x] Verify Android → HiveMQ publishing

**Done when:** Android publishes reliably to broker

---

##  Sprint 5 – ESP8266 Firmware
**Goal:** React to MQTT commands and control pump

- [x] Connect ESP8266 to Wi-Fi
- [x] Connect ESP8266 to HiveMQ Cloud (TLS)
- [x] Subscribe to command topic (`plant/pump/on`)
- [x] Drive relay / pump ON
- [x] Implement timed auto-OFF logic
- [x] Publish pump state (`plant/pump/state`)
- [x] Add reconnect handling

**Done when:** ESP reacts correctly to MQTT commands

---

##  Sprint 6 – End-to-End Integration
**Goal:** Full system operation

- [x] Android publishes ON command
- [x] ESP receives command
- [x] Pump activates
- [x] Pump auto-shuts off after timeout
- [x] State message published back to broker
- [x] Observe state in HiveMQ Web Client

**Done when:** Complete Android → Pump → State loop works

---

# Sprint 7 – Android App (MQTT Subscriber & Status UI)
**Goal:** Receive MQTT messages and reflect real device state in the UI

### MQTT Subscription & Handling
- [x] Subscribe to pump status topic (`plant/pump/status`)
- [x] Subscribe to device presence topic (`plant/device/online`)
- [x] Verify retained messages are received on app startup
- [x] Parse incoming payloads (`IDLE`, `RUNNING`, `DONE`, `ONLINE`, `OFFLINE`)
- [x] Log received messages for debugging
- [x] Handle messages arriving on background thread safely

### UI State Management
- [x] Add device status indicator (icon or dot)
- [x] Display device **ONLINE / OFFLINE** state
- [x] Display pump status text (`Idle`, `Running`, `Completed`)
- [x] Ensure UI updates occur on main thread
- [ ] Handle “unknown” state on first launch
- [ ] Prevent stale UI state after app reconnect

### Connection Awareness
- [x] Detect Android MQTT client connected/disconnected state
- [ ] Show “Connecting…” state when broker is unavailable
- [ ] Differentiate **broker offline vs ESP offline**
- [ ] Gracefully handle broker reconnects

### Validation
- [x] Verify ESP ONLINE message updates UI
- [x] Verify ESP OFFLINE (LWT) updates UI
- [x] Verify pump RUNNING → DONE transitions
- [ ] Restart app and confirm retained state restores UI
- [x] Disconnect ESP Wi-Fi and confirm OFFLINE indicator

**Done when:**  
Android app accurately reflects **real ESP presence and pump state** in real time and after reconnects.

## Sprint 8 – Watering History (Persistence + UI)
**Goal:** Record and display recent watering events and last-watered time.

### Implementation
- [x] Create in-memory history list (max N events)
- [x] Load saved history from SharedPreferences on app start
- [x] On pump status `DONE`, create a new history entry (timestamp)
- [x] Insert newest entry at top of list
- [x] Trim history list to max N entries
- [x] Save updated history back to SharedPreferences
- [x] Update UI: Last watered + History display


**Done when:** History updates on `DONE`, persists after app restart, and displays correctly.


## Sprint – Moisture Sensor Integration (ESP8266 + Android)

**Goal:**  
Integrate a soil moisture sensor into the system, publish readings via MQTT, and display moisture status in the Android app.

---

## Hardware Implementation

- [x] Identify moisture sensor pins (VCC, GND, SIGNAL)
- [x] Solder wires from moisture sensor to PCB
- [x] Verify correct power voltage for sensor
- [x] Confirm common ground between ESP8266 and sensor
- [x] Inspect solder joints and wire strain relief

**Done when:** Sensor is physically connected and stable on the board

---

## ESP8266 – Sensor Reading & Calibration

- [x] Select ADC pin for moisture sensor input
- [x] Read raw analog values from sensor
- [x] Test sensor in dry soil and record ADC value
- [x] Test sensor in wet soil and record ADC value
- [x] Determine usable moisture range (min/max)
- [x] Implement `map()` conversion to percentage (0–100%)
- [x] Constrain mapped values to valid range
- [x] Log raw and mapped values for debugging

**Done when:** Sensor readings are stable and mapped correctly

---

## ESP8266 – MQTT Publishing

- [x] Define MQTT topic for moisture data (e.g. `plant/moisture`)
- [x] Create non-blocking publish method
- [x] Convert moisture value to string payload
- [x] Publish moisture value at fixed interval 
- [x] Ensure publish does not block pump control logic
- [x] Verify messages appear in HiveMQ Web Client

**Done when:** Moisture values publish reliably to broker

---

## Android App – MQTT Subscription

- [x] Subscribe to moisture topic in MQTT helper
- [x] Receive moisture payloads from broker
- [x] Parse moisture value from payload
- [x] Handle invalid or missing values safely
- [x] Log received values for debugging

**Done when:** App receives moisture readings consistently

---

## Android App – UI Integration

- [x] Add TextView for moisture percentage
- [x] Update moisture TextView on message received
- [x] Ensure UI updates occur on UI thread
- [x] Confirm existing pump UI remains unaffected

**Done when:** Moisture value is visible and updates in real time

## Sprint 8 – Auto-Watering Iteration 
**Goal:** Fully working auto-watering with retained config, non-blocking pump control, 24h cooldown, and safe/verified behaviour end-to-end.

### App (Android)
- [x] Add switch + threshold slider listeners to publish config
- [x] Publish auto config to `plant/auto/config` as **retained**
- [x] Add small UI hints: “Auto ON/OFF”, “Threshold: X%”


### Firmware (ESP8266)
- [x] Subscribe to `plant/auto/config` on MQTT connect/reconnect
- [x] Implement config parsing (`enabled, threshold, durationMs, cooldownMin, hyst, maxPerDay`)
- [x] Store parsed values in config struct (`cfg`)
- [x] Update pump timer to use `cfg.durationMs` (not hardcoded `PUMP_MS`)
- [x] Implement auto decision function `maybeAutoWater(moisturePct)`
- [x] Add latch/hysteresis reset logic (prevents repeat watering while still low)
- [x] Enforce **24h cooldown** (millis-based for now)
- [x] Keep pump control **non-blocking** (no delays)
- [x] Add serial debug prints for: config received, moisture reading, auto decision reasons (temporary)

### Calibration + Behaviour Tuning
- [x] Log and confirm RAW → % mapping direction (dry vs wet)
- [x] Collect 3 baseline readings: air / dry soil / wet soil
- [x] Set final `RAW_DRY` and `RAW_WET` calibration values
- [x] Verify threshold behaviour: watering triggers only when `% < threshold`

### Integration Testing (HiveMQ + Real Hardware)
- [x] Test manual watering still works (button → pump runs → status updates)
- [x] Test auto watering triggers on low moisture (within next read interval)
- [x] Confirm cooldown prevents repeated watering within 24h
- [x] Confirm latch/hysteresis prevents rapid retrigger while moisture stays low
- [x] Regression check: online/offline LWT still correct


##  Sprint 10 – Documentation & Portfolio Polish
**Goal:** Make project portfolio-ready

- [] Document lessons learned
- [ ] Add system block diagram
- [ ] Document MQTT topic structure
- [ ] Add test evidence screenshots
- [ ] Clean up README
- [ ] Final review for public GitHub


-

