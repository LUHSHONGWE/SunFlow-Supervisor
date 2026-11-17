# OptiCharge-Sync: PV Supervisory Dashbo
 Optimizing Battery Charging while Maximizing load power utilisation.

An embedded supervisory control system designed to optimize battery charging, estimate State of Charge (SOC), and maximize Photovoltaic (PV) energy utilization under varying load conditions. 

## 📖 Project Overview
Conventional Maximum Power Point Tracking (MPPT) systems extract maximum instantaneous power from a solar array, but often ignore the electrochemical limits of the battery and fluctuating load demands. This results in inefficient energy utilization and accelerated battery degradation. 

**SunFlow Supervisor** acts as an intelligent layer above standard MPPT hardware. It utilizes a dual-microcontroller architecture (Arduino Nano + ESP32) to monitor the PV source, battery health, and load demand in real-time. The system enforces a "Battery-Health-First" strategy, dynamically routing excess PV power to a secondary load (Auxiliary Load) and providing actionable feedback via a custom web dashboard.

## ⚙️ System Architecture & Hardware
*   **Data Acquisition:** Arduino Nano (ATmega328P)
*   **Web Dashboard & Visualization:** ESP32 (Wi-Fi enabled)
*   **Power Stage (Charging):** BDD DC/DC Buck/MPPT Solar Charging Module (5A)
*   **Power Stage (Load):** XL4015 DC-DC Buck Converter (Stepping down to a stable 12V)
*   **Energy Storage:** 4S Lithium-Ion Battery Pack (14.8V Nominal) with 15A BMS
*   **Sensors:**
    *   3x 0-25V Voltage Sensors (Solar Input, Main Load Output, Battery SOC Estimation)
    *   2x ACS712 Hall-Effect Current Sensors (Solar Input, Main Load)
    *   1x 10k NTC Thermistor (Battery Thermal Monitoring)
*   **Switching:** 5V SPDT Relay Module (Hardware Power Routing)

## 🚀 Key Engineering Features
*   **Fractional Open-Circuit Voltage (FOCV) Advisory:** Calculates the optimal MPP setpoint (\(V_{mpp} = 0.76 \times V_{oc}\)) and displays recommendations to the user for manual MPPT tuning, bypassing the need for expensive digital potentiometers.
*   **Battery State of Charge (SOC) Estimation:** Utilizes a dedicated voltage sensor to monitor the battery bus. The system maps the measured voltage against a 4S Lithium charge curve to estimate the SOC percentage.
*   **Intelligent Load Routing (Hysteresis Control):** Uses a relay to route power between the Main Load and an Auxiliary Load. When the battery reaches a safe charging threshold (e.g., >16.5V) and solar is available, excess energy is diverted to the Aux load. Hysteresis logic prevents rapid relay chattering during voltage sag.
*   **Thermal Protection:** Monitors battery temperature. If the temperature exceeds safe limits (>45°C), the system recommends terminating the charge to prevent thermal runaway.
*   **Real-Time Web Dashboard:** An ESP32-hosted web interface displays live graphs of Solar Power, Load Power, Battery Voltage, SOC, and Current Operating Mode.

## 🧠 Supervisory Logic Flow
1.  **Normal Mode:** Battery is charging. The relay is in the Normally Closed (NC) state, powering the primary 12V load via the buck converter.
2.  **Excess Power Mode:** When the battery hits the full-voltage threshold and the sun is out, the system switches to the Normally Open (NO) contact, routing excess solar energy to the auxiliary load.
3.  **Protection Mode:** If thermal limits are breached, a visual warning is displayed on the dashboard, prompting the user to manually disconnect the charging path.

## 📂 Repository Structure
*   `/Arduino_Nano`: Contains the core C++ firmware for sensor polling, SOC calculation, and relay logic.
*   `/ESP32_Dashboard`: Contains the ESP32 code for UART data parsing and the Web UI (HTML/CSS/JS embedded).
*   `/Documentation`: Circuit schematics, block diagrams, and the final research article.
*   `LICENSE`: MIT License.

## 🛠️ Installation & Setup
1.  **Hardware Wiring:** Refer to the `Documentation/Circuit_Diagram.pdf`. Ensure all sensors share a common ground.
2.  **Calibration:** On startup, ensure zero current is flowing through the ACS712 sensors for auto-calibration.
3.  **Firmware Upload:** Upload the Arduino Nano code first, then the ESP32 code. Ensure the RX/TX pins are crossed and grounded correctly.
4.  **Power Up:** Connect the battery (via BMS) and verify the ESP32 dashboard is receiving serial data.

## 🔮 Future Improvements
*   **Automated MPPT:** Replace the manual potentiometer on the BDD module with a high-voltage digital potentiometer (e.g., MCP41HV51) to allow the Arduino to adjust the MPP automatically via SPI.
*   **Advanced SOC Estimation:** Implement a coulomb-counting method using the current sensor for more accurate SOC tracking than voltage-based estimation.
*   **PCB Design:** Migrate the breadboard prototype to a custom PCB to reduce electrical noise and improve reliability.

