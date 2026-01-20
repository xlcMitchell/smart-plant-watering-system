# Lessons Learned – Smart Plant Watering System

This document captures key lessons learned during the design, implementation, testing, and debugging of the ESP8266-based smart plant watering system and Android application.

---

## 1. Power Delivery Is Critical for IoT Devices
**Lesson:**  
ESP8266 stability is highly sensitive to power quality, especially during Wi-Fi transmission and when driving external loads.

**What Happened:**  
Using a 9V PP3 battery and linear regulation caused voltage sag, regulator overheating, and repeated ESP resets during pump activation.

**What I Learned:**  
- ESP8266 requires high transient current  
- Linear regulators are inefficient with high input voltages  
- Brownouts can appear as firmware or timing bugs  

**What I Would Do Differently:**  
- Design the power architecture first  
- Use a buck converter and separate power rails for logic and actuators  
- Validate power stability under load early  

---

## 2. Hardware Issues Can Look Like Software Bugs
**Lesson:**  
Unstable hardware can produce symptoms that closely resemble software logic errors.

**What Happened:**  
MQTT disconnects, missed messages, and inconsistent pump timing were initially suspected to be firmware issues but were ultimately caused by power instability.

**What I Learned:**  
- Hardware stability must be verified before deep software debugging  
- Serial logs alone do not reveal power-related issues  

**What I Would Do Differently:**  
- Measure supply voltages under load earlier  
- Add reset and brownout indicators during testing  

---

## 3. Persistence Bugs Are Easy to Miss
**Lesson:**  
Correct UI behaviour does not guarantee data persistence.

**What Happened:**  
Watering history updated correctly in the UI but was not saved across app restarts due to a missing `apply()` / `commit()` call in SharedPreferences.

**What I Learned:**  
- In-memory correctness ≠ persistent correctness  
- Persistence logic must be explicitly tested  

**What I Would Do Differently:**  
- Add persistence-specific test cases earlier  
- Log storage writes during development  

---

## 4. Timing Expectations Matter in Automated Systems
**Lesson:**  
Automated actions may feel delayed if they are tied to scheduled sensor readings.

**What Happened:**  
Auto-watering only triggered after the next moisture reading interval, which initially appeared as a delay.

**What I Learned:**  
- Understanding system timing is essential  
- Perceived latency can be expected behaviour  

**What I Would Do Differently:**  
- Document timing behaviour clearly  
- Consider immediate checks when configuration changes  

---

## 5. Sensor Behaviour Requires Empirical Validation
**Lesson:**  
Sensor assumptions must be validated with real data.

**What Happened:**  
Moisture percentage mapping initially appeared inverted until raw ADC values were logged and correlated with soil conditions.

**What I Learned:**  
- Raw data logging is essential  
- Mapping functions must be validated experimentally  

**What I Would Do Differently:**  
- Log raw and mapped values from the start  
- Average multiple readings to reduce noise  

---

## 6. Retained MQTT Messages Improve Robustness
**Lesson:**  
Retained MQTT messages simplify state recovery after reconnects.

**What Happened:**  
Using retained messages allowed both the ESP8266 and Android app to recover state automatically after reconnecting.

**What I Learned:**  
- Retained topics are ideal for configuration and status  
- Command topics should generally not be retained  

**What I Would Do Differently:**  
- Explicitly define retained vs non-retained topics in design documentation  

---

## 7. Structured Testing Saves Time
**Lesson:**  
Formal test cases and logs accelerate debugging and reduce guesswork.

**What Happened:**  
Structured test logs made it clear that failures were hardware-related rather than firmware-related.

**What I Learned:**  
- Writing tests improves clarity and debugging efficiency  
- Documentation helps isolate root causes  

**What I Would Do Differently:**  
- Start test documentation earlier  
- Add regression tests immediately after fixes  

---

## 8. Version Control Enables Safe Experimentation
**Lesson:**  
Branching allows experimentation without risking stable code.

**What Happened:**  
Using a separate Git branch for auto-watering enabled rapid iteration and rollback when logic became complex.

**What I Learned:**  
- Feature branches reduce fear of breaking working code  
- Even solo projects benefit from proper Git workflows  

**What I Would Do Differently:**  
- Create feature branches earlier  
- Commit smaller, more focused changes  

---

## Overall Reflection
This project reinforced the importance of:
- Power-first hardware design  
- Clear separation between hardware and software faults  
- Explicit testing of persistence and timing behaviour  
- Empirical validation over assumptions  

The integration of embedded firmware, MQTT networking, Android UI development, and real-world hardware debugging provided a realistic end-to-end IoT development experience.
