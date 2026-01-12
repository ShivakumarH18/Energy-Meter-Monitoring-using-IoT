# Energy-Meter-Monitoring-Using-IoT
Final year project – Energy Meter Monitoring using IoT



# Energy Meter Monitoring Using IoT

## 📌 Project Overview
Electricity is an essential part of daily life in homes and industries. Traditional manual meter reading is time-consuming and may cause billing errors. With the help of Internet of Things (IoT), it is possible to monitor electricity usage remotely and in real time.  
This project integrates an energy meter with a microcontroller and Wi-Fi module to send live energy consumption and cost data to an online platform, which users can access using mobile or web applications.

## ❗ Problem Statement & Motivation
Manual electricity meter reading:
- Takes more time
- Can cause human errors
- Does not allow real-time monitoring  
Consumers cannot easily verify their electricity usage or bills. Because of this, energy is often wasted.  
Hence, there is a need for an automated system that provides real-time and remote access to electricity readings.

## 🎯 Objectives
- To design an IoT-based real-time energy monitoring system.
- To eliminate manual meter reading.
- To provide transparency in electricity billing.
- To help users track and optimize electricity usage.
- To enable remote access to energy data.

## 📚 Literature Review
Several researchers have developed IoT-based smart energy meters for monitoring, billing, and energy optimization.  
Some systems also include energy theft detection and load control.  
From these studies, we learned about:
- Pulse-based energy measurement
- Sensor isolation using optocouplers
- Cloud-based data monitoring  
These works helped us in selecting components and designing an efficient system.

## 🧩 Block Diagram Description
The system consists of:
- AC Supply
- Energy Meter
- Optocoupler (4N35)
- Arduino Microcontroller
- ESP8266 Wi-Fi Module
- LCD Display
- Push Buttons
- Voltage Regulators (5V and 3.3V)

The energy meter measures power usage and produces pulses. These pulses are isolated using an optocoupler and then processed by the microcontroller. Data is displayed on LCD and sent to cloud using Wi-Fi.

## 🔩 Components Used
- Energy Meter: Measures electricity consumed.
- Arduino Microcontroller: Controls system and processes pulses.
- ESP8266 Wi-Fi Module: Sends data to online server.
- LCD Display: Shows real-time units and cost.
- Optocoupler (4N35): Provides electrical isolation and safety.
- Voltage Regulators: Provide required power supply.
- Push Buttons: Used for reset, tariff selection, and system control.

## ⚙️ Methodology
1. Energy meter generates pulses based on power consumption.
2. Optocoupler converts meter LED blink into safe digital signal.
3. Microcontroller counts pulses and calculates units (kWh).
4. Cost is calculated based on tariff selected.
5. Data is sent to Wi-Fi module and uploaded to cloud.
6. LCD displays units, pulse count, and total cost locally.
7. User can monitor energy usage remotely via web or mobile.

## ✅ Outcomes
- Automated electricity monitoring system.
- Real-time energy tracking using IoT.
- Improved transparency in billing.
- Reduction in manual meter reading work.
- Better energy management and awareness.

## Applications
- Smart homes
- Industrial energy monitoring
- Electricity billing systems
- Energy auditing
- Smart grid systems

 ## Future Enhancements
- Mobile application integration
- Automatic billing system
- Alerts for high energy usage
- Theft detection features
- AI-based energy prediction

## 📖 References
- Munoz et al., Sensors Journal, 2022.
- Kolawole et al., British Journal of Computer Networking, 2025.
- Bhoite & Kanse, IoT-Based Electricity Theft Detection, 2025.
- T. Karthick et al., Energy Reports, 2021.
- Vishakha Yadav, IoT Energy Monitoring, 2023.

## 📌 Conclusion
The IoT-based energy meter monitoring system enables real-time tracking of electricity usage with better accuracy. It reduces manual errors and gives users instant access to consumption data. The system supports efficient energy management and provides a cost-effective and smart solution for electricity monitoring.
