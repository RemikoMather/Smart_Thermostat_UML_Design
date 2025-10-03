# UML Diagram Descriptions

This document provides detailed textual descriptions of the three UML diagrams for the Smart Thermostat System. These descriptions can be used to understand the diagrams without rendering the PlantUML files.

---

## 1. Use Case Diagram Description

### Actors

1. **Homeowner** - Primary end-user of the system
2. **Installer** - Professional or DIY installer who sets up the device
3. **HVAC System** (<<system>>) - External heating/cooling equipment
4. **Weather Service** (<<external>>) - Third-party weather data provider

### Use Cases (within System Boundary)

**Primary User Functions:**
1. Set Temperature
2. Create Schedule
3. View Energy Report
4. Set Away Mode
5. Update Firmware

**Installation & Configuration:**
6. Configure WiFi
7. Install Device
8. Calibrate Sensors

**System Functions:**
9. Receive Weather Updates
10. Control HVAC
11. Monitor Energy Usage
12. Send Alerts
13. Authenticate User
14. Adjust Temperature Automatically

### Relationships

**Actor Associations:**
- Homeowner connects to: Set Temperature, Create Schedule, View Energy Report, Set Away Mode, Update Firmware
- Installer connects to: Configure WiFi, Install Device, Calibrate Sensors
- HVAC System connects to: Control HVAC
- Weather Service connects to: Receive Weather Updates

**<<include>> Relationships:**
- Set Temperature includes Authenticate User
- Create Schedule includes Authenticate User
- View Energy Report includes Authenticate User
- Set Away Mode includes Authenticate User
- Set Temperature includes Control HVAC
- Create Schedule includes Control HVAC
- Set Away Mode includes Control HVAC

**<<extend>> Relationships:**
- Adjust Temperature Automatically extends Set Temperature
- Adjust Temperature Automatically extends Receive Weather Updates
- Send Alerts extends Monitor Energy Usage

### Visual Layout
The system boundary is clearly marked as "Smart Thermostat System" with all use cases contained within. Actors are positioned outside the boundary, with human actors (Homeowner, Installer) on the left side and system actors (HVAC System, Weather Service) on the right side.

---

## 2. Sequence Diagram Description

### Scenario
"User Sets Temperature via Mobile App" - showing the complete workflow from user input to HVAC activation and target achievement.

### Lifelines (Participants)

1. **Homeowner** (Actor)
2. **:MobileApp** (Object)
3. **:CloudServer** (Object)
4. **:ThermostatController** (Object)
5. **:TemperatureSensor** (Object)
6. **:HVACUnit** (Object)

### Message Sequence

**Phase 1: Authentication and Initial Display**
1. Homeowner → MobileApp: "Open app"
2. MobileApp → CloudServer: "Request authentication"
3. CloudServer → MobileApp: "Authentication token" (return)
4. MobileApp → Homeowner: "Display current temperature and controls"

**Phase 2: Temperature Setting Request**
5. Homeowner → MobileApp: "Set temperature to 72°F"
6. MobileApp → MobileApp: "Validate input" (self-call)
7. MobileApp → CloudServer: "Send temperature update (72°F, timestamp, device_id)"
8. CloudServer → CloudServer: "Log request" (self-call)
9. CloudServer → ThermostatController: "Forward temperature command (72°F)"

**Phase 3: Thermostat Processing**
10. ThermostatController → ThermostatController: "Authenticate command" (self-call)
11. ThermostatController → TemperatureSensor: "Read current temperature"
12. TemperatureSensor → ThermostatController: "Return 68°F" (return)
13. ThermostatController → ThermostatController: "Calculate required action (Need heating: 72°F > 68°F)" (self-call)

**Phase 4: HVAC Activation**
14. ThermostatController → HVACUnit: "Send command (HEAT, target: 72°F)"
15. HVACUnit → HVACUnit: "Start heating system" (self-call)
16. HVACUnit → ThermostatController: "Acknowledge command" (return)
17. ThermostatController → CloudServer: "Send status update (heating, target: 72°F)"
18. CloudServer → MobileApp: "Push notification (heating started)"
19. MobileApp → Homeowner: "Display 'Heating to 72°F'"

**Phase 5: Monitoring Loop**
20. **Loop [Every 30 seconds until target reached]:**
    - ThermostatController → TemperatureSensor: "Read current temperature"
    - TemperatureSensor → ThermostatController: "Return temperature" (return)
    - ThermostatController → CloudServer: "Send temperature update"
    - CloudServer → MobileApp: "Push current temperature"
    - MobileApp → Homeowner: "Update display"

**Phase 6: Target Achievement**
21. ThermostatController → TemperatureSensor: "Read current temperature"
22. TemperatureSensor → ThermostatController: "Return 72°F" (return)
23. ThermostatController → HVACUnit: "Send command (IDLE)"
24. HVACUnit → HVACUnit: "Stop heating" (self-call)
25. HVACUnit → ThermostatController: "Acknowledge" (return)
26. ThermostatController → CloudServer: "Target reached notification"
27. CloudServer → MobileApp: "Push notification (Target temperature reached)"
28. MobileApp → Homeowner: "Display '72°F - Target reached'"

### Key Features
- Activation bars show when each object is actively processing
- Synchronous messages (solid arrows) for request-response interactions
- Asynchronous messages for push notifications
- Self-calls (reflexive arrows) for internal processing
- Loop fragment for continuous monitoring
- Clear temporal ordering from top to bottom

---

## 3. Deployment Diagram Description

### Hardware Nodes

**1. Cloud Server** (<<execution environment>>)
- Contains artifacts:
  - ThermostatAPI.jar
  - UserDatabase
  - AnalyticsService.jar
  - User Data Store (database)
- Internal connections: API connects to UserDatabase, AnalyticsService connects to DataStore

**2. User's Smartphone** (<<device>>)
- Contains execution environment: iOS/Android OS
  - Contains artifact: MobileApp.apk/.ipa

**3. Smart Thermostat** (<<device>>)
- Contains execution environment: Embedded Linux
  - Contains artifacts:
    - ThermostatFirmware.jar
    - SensorDriver.so
    - WiFiModule.so
- Contains hardware components:
  - Temperature Sensor (<<hardware>>)
  - Humidity Sensor (<<hardware>>)
  - Display Screen (<<hardware>>)
  - WiFi Chip (<<hardware>>)
- Internal connections:
  - Firmware connects to SensorDriver and WiFiModule
  - SensorDriver connects to Temperature Sensor and Humidity Sensor
  - WiFiModule connects to WiFi Chip

**4. HVAC Unit** (<<device>>)
- Contains hardware components:
  - Heating System (<<hardware>>)
  - Cooling System (<<hardware>>)
  - Fan System (<<hardware>>)
- Contains artifact: HVACController.hex
- Internal connections: HVACController connects to all three subsystems

**5. Home WiFi Router** (<<device>>)
- Contains hardware component:
  - WiFi Access Point (<<hardware>>)

**6. Weather Service API** (<<external system>>)
- Cloud-based external system
- Contains artifact: WeatherData API

### Communication Paths

1. **User's Smartphone ↔ Home WiFi Router**
   - Protocol: WiFi (802.11 b/g/n/ac)

2. **Smart Thermostat ↔ Home WiFi Router**
   - Protocol: WiFi (802.11 b/g/n)

3. **Home WiFi Router ↔ Cloud Server**
   - Protocol: HTTPS (Internet)

4. **Smart Thermostat ↔ HVAC Unit**
   - Protocol: C-Wire/24VAC (Control Signal)

5. **Cloud Server ↔ Weather Service API**
   - Protocol: HTTPS/REST API

### Annotations (Notes)

**Smart Thermostat Note:**
- Primary Hub:
  - Processes temperature data
  - Controls HVAC directly
  - Syncs with cloud
  - Low power consumption

**Cloud Server Note:**
- Backend Services:
  - User authentication
  - Schedule storage
  - Energy analytics
  - Mobile app API
  - Firmware updates

**User's Smartphone Note:**
- User Interface:
  - Remote control
  - Schedule management
  - Energy reports
  - Notifications

### Key Architectural Features
- Distributed system with edge processing (thermostat) and cloud services
- Standard WiFi connectivity eliminating need for proprietary gateways
- Direct HVAC control via industry-standard wiring
- External API integration for enhanced functionality
- Multi-platform mobile app support (iOS/Android)
