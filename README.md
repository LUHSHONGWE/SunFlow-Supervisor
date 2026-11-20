# SunFlow-Supervisor: PV Supervisory Dashbo
 Optimizing Battery Charging while Maximizing load power utilisation.

An embedded supervisory control system designed to optimize battery charging, estimate State of Charge (SOC), and maximize Photovoltaic (PV) energy utilization under varying load conditions. 

📖 Project Overview
Conventional Maximum Power Point Tracking (MPPT) systems are designed to extract maximum instantaneous power from a solar array. However, they often operate blindly—ignoring the electrochemical limits of the battery and the fluctuating demands of the connected load. This mismatch results in inefficient energy utilization, accelerated battery degradation, and potential thermal runaway.

OptiCharge-Sync (formerly SunFlow Supervisor) is an embedded supervisory control system designed to solve this problem. It acts as an intelligent "brain" sitting above standard MPPT hardware. Utilizing a dual-microcontroller architecture, the system monitors the PV source, battery health, and load demand in real-time. It enforces a Battery-Health-First strategy, dynamically routing excess PV power to an auxiliary load via relay switching, and provides actionable feedback through a custom, locally hosted web dashboard.

🔗 Live Dashboard UI (Web Showcase)
View the Web UI: https://luhshongwe.github.io/SunFlow-Supervisor/

Access PIN: 1234
(Note: Because the live hardware telemetry requires a local connection to the ESP32, this link serves as a front-end UI/UX showcase. Use the PIN to bypass the security lockout and view the dashboard interface).

🔬 The Engineering Problem & Methodology
The Problem: Standard MPPT systems treat the solar panel, battery, and load as independent entities. When the battery is full, MPPT systems continue pushing voltage, causing overcharging. When the load demands more power than the sun provides, the system lacks intelligent load prioritization.

The Solution:

Data Acquisition: An Arduino Uno polls 3 voltage sensors (Solar, Battery, Load), 2 ACS712 current sensors (Charge, Load), and a 10k NTC thermistor. It calculates real-time power and transmits the data via UART.

Supervisory Logic: An ESP32 receives the telemetry, applies hysteresis logic (to prevent relay chattering), and manages the safety thresholds (Thermal and SOC limits).

Hardware Routing: Based on the ESP32's logic, a 5V SPDT relay physically switches power. By default, power flows to the Main 12V Load (Normally Closed). When the battery hits the full-charge threshold (>16.5V), the ESP32 triggers the relay to switch power to the Auxiliary Pool Pump Load (Normally Open), maximizing PV utilization.

Web Visualization: The ESP32 hosts a local asynchronous web server, displaying live JSON data on a responsive dashboard.

⚙️ System Architecture & Hardware Stack
Component	Specification	Role
Data Acquisition	Arduino Uno (ATmega328P)	Sensor polling, ADC conversion, power calculation
Web Server & Logic	ESP32 (Wi-Fi enabled)	UART parsing, Supervisory logic, Relay control, HTML hosting
Charging Stage	BDD DC/DC Buck/MPPT Solar Module (5A)	Steps down 21V Solar to charge the 4S Battery
Load Stage	XL4015 DC-DC Buck Converter	Steps down fluctuating battery voltage to a stable 12V
Energy Storage	4S Lithium-Ion Pack (14.8V Nominal)	Power storage with 15A BMS for cell protection
Sensors	3x 0-25V Voltage Divider, 2x ACS712 (20A), 1x 10k NTC	Telemetry and thermal monitoring
Switching	5V SPDT Relay Module	Hardware power routing (NC to NO)
🚀 Key Engineering Features
Fractional Open-Circuit Voltage (FOCV) Advisory: Because the MPPT module uses manual potentiometers, the dashboard calculates the optimal MPP setpoint (
V
m
p
p
=
0.76
×
V
o
c
V 
mpp
​
 =0.76×V 
oc
​
 ) and gives the user real-time recommendations to adjust the hardware.

Intelligent Relay Routing (Hysteresis Control): The ESP32 switches the relay to the Aux Load when the battery is full. It utilizes hysteresis logic: it only switches back to the main load when the battery drops below 14.0V, preventing rapid relay clicking.

Web-Based Relay Override: The web dashboard includes a manual override button allowing the user to force the relay to switch to the Aux Load, bypassing automatic FOCV logic for testing.

Thermal Protection: If the NTC thermistor detects a battery temperature exceeding 45°C, the system enters "PROTECT" mode and cuts the charge path.

Security Lockout: The web dashboard features a PIN-protected login screen with a 3-attempt lockout (30-second timeout) to prevent unauthorized access.

CORS-Bypassed Local Server: The ESP32 is configured with Access-Control-Allow-Origin: * to allow secure cross-origin communication between the hardware and web clients.

📂 Repository Structure
/Arduino_Sensors: C++ firmware for the Arduino Uno (Sensor calibration, ADC polling, UART transmission).

/ESP32_Supervisor: C++ firmware for the ESP32 (UART RX, Logic control, Relay switching, Async Web Server).

/Web_Dashboard: Contains the standalone index.html file for the GitHub Pages showcase.

/Documentation: Circuit schematics, block diagrams, and final research paper.

🛠️ Installation & Setup (For Local Hardware)
Hardware Wiring: Ensure all sensors and microcontrollers share a common ground. The Arduino TX (Pin 1) must connect to the ESP32 RX2 (GPIO 16).

Calibration: Power up the system with zero current flowing through the ACS712 sensors. The Arduino auto-calibrates the zero-offset voltage in setup().

Firmware Upload: Upload the Arduino code first, then the ESP32 code.

Network Connection: The ESP32 will connect to the designated Wi-Fi hotspot and print its local IP address to the Serial Monitor (115200 baud). Type this IP into your browser to view the dashboard.

🔮 Future Improvements & Roadmap
If I were to continue developing this project, the next engineering phases would include:

Automated MPPT Tracking: Replace the manual potentiometer on the BDD module with a digital potentiometer (e.g., MCP41HV51) to allow the ESP32 to automatically adjust the MPP via SPI, eliminating the need for human intervention.

Advanced SOC Estimation: Transition from voltage-based SOC estimation to a Coulomb-counting method using the charge current sensor, providing far more accurate battery tracking.

Cloud Integration: Implement MQTT or WebSockets to push data to a cloud service (like AWS IoT or ThingSpeak), allowing users to monitor their solar array from anywhere in the world, not just the local Wi-Fi network.

PCB Fabrication: Migrate the prototype from breadboards to a custom 2-layer PCB to eliminate parasitic capacitance, electrical noise, and loose jumper wires.
