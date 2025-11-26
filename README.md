# RT-MoniX-Real-Time-Embedded-Monitoring-Framework



Introduction
---

This project demonstrates a multi-sensor synchronization system running on an ESP32 with FreeRTOS, simulated inside the Wokwi online environment.
The system reads data from two sensors:

1 LDR light sensor (connected through ADC)

2 DHT22 temperature/humidity sensor

However, due to Wokwi’s current limitations on C-based DHT22 control libraries, the official DHT22 driver cannot be used directly. Because Wokwi does not allow modification of its internal dependency files, the DHT22 driver cannot be adapted to FreeRTOS timing requirements or ESP32 GPIO timing.
As a result, the DHT22 readings in this project are simulated using random numbers to mimic temperature and humidity values.

Despite this limitation, the project successfully demonstrates:

A synchronized multi-sensor reading system

- FreeRTOS task-to-task communication

- FreeRTOS task notification mechanism

- ADC reading from LDR

- Coordination via a central timer task

- Passing synchronized sensor data through a queue

This project is useful as a learning model for RTOS task design, sensor synchronization, and embedded system architecture.


Wokwi Limitation: Why DHT22 Cannot Be Used Normally
---
The DHT22 sensor normally requires extremely precise microsecond-level timing.
Wokwi provides a pre-built C driver for DHT sensors, but:

- The header and driver code used by Wokwi cannot be modified.

- ESP-IDF + FreeRTOS introduces task scheduling delays that break the required DHT timing.

- Attempting to write a custom DHT22 driver fails because accurate microsecond control is not achievable under FreeRTOS on Wokwi.

- As a result, the DHT22 driver always fails to initialize or cannot return data.

Because of these constraints, the project uses a custom mock function to simulate DHT22 readings.


Simulated ranges:

- Temperature: 25.00°C ~ 25.99°C

- Humidity: 60.00% ~ 60.49%

The simulation includes a realistic delay (about 250 ms) to mimic an actual DHT reading cycle.


Project Overview
---
This project contains the following components:

(1) LDRReadTask

Reads LDR value via ADC1 Channel 0.

Waits for the coordinator’s notification, then performs a one-shot ADC read.

Sends the ADC value to the DHT task via task notification.


(2) DHTReadTask

Waits for a cycle notification from the coordinator.

Simulates DHT22 temperature/humidity data.

Receives LDR ADC data from LDRReadTask.

Combines all sensor data into a single structure.

Sends the combined data to a queue.


(3) TimerCoordinatorTask

High-priority task.

Triggers both sensor tasks every 5 seconds.

Ensures that readings occur synchronously each cycle.


(4) Queue (xCombinedDataQueue)

Holds the synchronized sensor output structure: temperature, humidity, and LDR ADC value.

Ready for future use (e.g., display, logging, MQTT, or GUI output).


System Behavior Summary
---
Every 5 seconds:

Coordinator task broadcasts a notification to LDR and DHT tasks.

LDRReadTask performs an ADC read.

DHTReadTask simulates DHT22 data.

DHTReadTask waits until LDRReadTask sends the ADC value.

Both results are synchronized and packaged into CombinedData_t.

The synchronized data is printed and pushed into the queue.

This architecture shows how multiple sensors can be cleanly synchronized under FreeRTOS without blocking or polling.



How to Run on Wokwi
---

- Upload all project files to your Wokwi ESP32 FreeRTOS project.

- Because this version does not depend on real DHT22 hardware timing, it will work on the simulator.

- Open the Serial Monitor to view synchronized sensor output every 5 seconds.

No hardware setup is required because the DHT is simulated.


Code File Explanation
---
The code implements the synchronized multi-sensor reading system described above. Key parts of the implementation include:

- GPIO configuration: sets up the DHT GPIO pin and ADC input pin for the LDR.
- ADC one-shot driver initialization: configures ADC1 Channel 0 (GPIO36/VP) for single-sample reads on demand.
- FreeRTOS task notification usage: uses direct-to-task notifications for lightweight, low-latency inter-task signalling between `LDRReadTask` and `DHTReadTask`.
- Random number simulation for DHT22: a mock routine simulates temperature and humidity values and includes a ~250 ms delay to approximate a real DHT22 read cycle.
- Queue-based data passing: combined sensor data is sent to `xCombinedDataQueue` for later consumption or processing.
- Structured sensor synchronization logic: a high-priority `TimerCoordinatorTask` triggers both sensor tasks at the configured sampling interval, ensuring synchronized readings each cycle.

Key definitions
---

- **DHT GPIO pin:** `GPIO4`
- **LDR ADC channel:** `ADC1 Channel 0` (pin `GPIO36` / `VP`)
- **Sampling interval:** `5 seconds`

Known Issues / Future Improvements
---

1. DHT22 cannot be used in real timing mode on Wokwi.
	- Cause: Wokwi's C-driver and the FreeRTOS scheduling environment do not allow the microsecond-precise timing DHT22 requires.
	- Workarounds / improvements:
	  - Use a JavaScript-based Wokwi part that emulates DHT timing logic inside the simulator.
	  - Prefer I2C or SPI sensors (e.g., SHT31 or BME280) for simulations under FreeRTOS, since those interfaces are more robust in virtualized environments.
	  - For physical hardware testing, run the code on a real ESP32 dev board and use a proven DHT22 driver adapted for ESP-IDF timing if necessary.

2. ADC reading may differ from real hardware.
	- Wokwi simulates ADC behavior approximately and values may not exactly match physical sensor readings.
	- Recommendation: validate ADC calibration on real hardware if precise lux/voltage readings are required.

3. Queue is currently reserved for future expansion.
	- At present, `xCombinedDataQueue` stores the synchronized sensor output, but the project does not include additional tasks that consume the queue.
	- Future work: add logger, display task, or networking task (MQTT/HTTP) to consume and forward queued data.

Conclusion
---

This Wokwi-based ESP32 + FreeRTOS project demonstrates how to:

- Implement multiple sensor reading tasks
- Synchronize sensor sampling using a coordinator task
- Pass composed sensor data through FreeRTOS queues
- Handle missing or unsupported hardware by using simulation techniques

Although Wokwi cannot support the DHT22 C-driver timing reliably, the simulated DHT module preserves the system architecture for educational and prototyping purposes. The project makes it straightforward to replace the mock DHT with a real I2C/SPI sensor or to test on physical hardware when precise timing and ADC accuracy are required.


