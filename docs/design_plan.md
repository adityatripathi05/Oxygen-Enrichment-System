# Oxygen Enrichment Control System Design_v2.1
## Architecture Plan

## Table of Contents
- [Oxygen Enrichment Control System Design\_v2.1](#oxygen-enrichment-control-system-design_v21)
  - [Architecture Plan](#architecture-plan)
  - [Table of Contents](#table-of-contents)
  - [1. System Overview](#1-system-overview)
  - [2. High-Level Design (HLD)](#2-high-level-design-hld)
    - [2.1 System Architecture](#21-system-architecture)
    - [2.2 Component Diagram](#22-component-diagram)
    - [2.3 Deployment Architecture](#23-deployment-architecture)
  - [3. Low-Level Design (LLD)](#3-low-level-design-lld)
    - [3.1 Hardware Components](#31-hardware-components)
    - [3.2 Software Components](#32-software-components)
      - [Microcontroller Firmware (ESP32)](#microcontroller-firmware-esp32)
      - [Zone Raspberry Pi Software](#zone-raspberry-pi-software)
      - [Central Raspberry Pi Software](#central-raspberry-pi-software)
    - [3.3 Database Schema](#33-database-schema)
    - [3.4 API Design](#34-api-design)
      - [Zone RPi API Endpoints](#zone-rpi-api-endpoints)
      - [Central RPi API Endpoints](#central-rpi-api-endpoints)
    - [3.5 Sensor Sampling Framework](#35-sensor-sampling-framework)
      - [Sampling Frequency Configuration](#sampling-frequency-configuration)
      - [Data Processing Pipeline](#data-processing-pipeline)
      - [Data Retention Policies](#data-retention-policies)
    - [3.6 Logging and Monitoring System](#36-logging-and-monitoring-system)
      - [Logging Levels](#logging-levels)
  - [4. Zone Operation Modes](#4-zone-operation-modes)
    - [4.1 Sensor-based Zone Operation](#41-sensor-based-zone-operation)
    - [4.2 Timer-based Zone Operation](#42-timer-based-zone-operation)
    - [4.3 Boost Mode Operation](#43-boost-mode-operation)
  - [5. Data Flow Diagrams](#5-data-flow-diagrams)
    - [5.1 Sensor Data Flow](#51-sensor-data-flow)
    - [5.2 Control System Flow](#52-control-system-flow)
    - [5.3 User Interaction Flow](#53-user-interaction-flow)
    - [5.4 Time-based Control Flow](#54-time-based-control-flow)
  - [6. User Management System](#6-user-management-system)
    - [6.1 Authentication System](#61-authentication-system)
    - [6.2 Technician Interface](#62-technician-interface)
    - [6.3 Guest Interface](#63-guest-interface)
    - [6.4 Zone Assignment Process](#64-zone-assignment-process)
  - [7. User Interface Design](#7-user-interface-design)
    - [7.1 Technician Dashboard](#71-technician-dashboard)
    - [7.2 Zone Setup Interface](#72-zone-setup-interface)
    - [7.3 Guest Control Interface](#73-guest-control-interface)
  - [8. Processing and Management Plan](#8-processing-and-management-plan)
    - [8.1 Processing Optimization](#81-processing-optimization)
    - [8.2 Data Management Strategy](#82-data-management-strategy)
    - [8.3 Current Infrastructure](#83-current-infrastructure)
  - [9. References](#9-references)

## 1. System Overview

The Oxygen Enrichment Control System is designed for vacation guest houses to monitor and manage oxygen levels in different zones. The system now supports both sensor-based and timer-based oxygen control:

1. **Sensor-based Zones**: Use oxygen sensors connected to ESP32 microcontrollers that communicate via Bluetooth with zone-specific Raspberry Pi units.
2. **Timer-based Zones**: Use predefined time slots to control oxygen supply where sensors cannot be installed.

Each zone has an oxygen supply unit controlled through Web Ethernet relays. The central Raspberry Pi collects data from all zones and provides system-wide monitoring and control.

**Key Features:**
- Real-time oxygen level monitoring (in sensor-based zones)
- Automated oxygen enrichment based on predefined thresholds or time slots
- Altitude equivalency calculation
- Zone-specific monitoring and control
- Centralized management interface for technicians
- Local touchscreen displays in each zone for guest control
- Boost mode for instant oxygen enrichment
- Configurable operation time slots for sensor-based zones
- Zone assignment system for guest house management
- Maintenance lock system for zones

## 2. High-Level Design (HLD)

### 2.1 System Architecture

```mermaid
graph TD
    subgraph "Sensor-based Zone"
        OS1[Oxygen Sensor] --> ESP1[ESP32 Microcontroller]
        ESP1 -->|BLE| RPI1[Zone Raspberry Pi]
        RPI1 -->|Ethernet| ER1[Ethernet Relay]
        ER1 --> OU1[Oxygen Unit]
        RPI1 -->|Ethernet| SW[Ethernet Switch]
        RPI1 --- TD1[7-inch Touchscreen Display]
    end

    subgraph "Timer-based Zone"
        RPI2[Zone Raspberry Pi] -->|Ethernet| ER2[Ethernet Relay]
        ER2 --> OU2[Oxygen Unit]
        RPI2 -->|Ethernet| SW
        RPI2 --- TD2[7-inch Touchscreen Display]
    end

    subgraph "Zone N"
        OSN[Oxygen Sensor] --> ESPN[ESP32 Microcontroller]
        ESPN -->|BLE| RPIN[Zone Raspberry Pi]
        RPIN -->|Ethernet| ERN[Ethernet Relay]
        ERN --> OUN[Oxygen Unit]
        RPIN -->|Ethernet| SW
        RPIN --- TDN[7-inch Touchscreen Display]
    end

    SW -->|Ethernet| CRPI[Central Raspberry Pi]
    CRPI --> DB[(SQLite Database)]
    CRPI --> WI[Web Interface]
    CRPI --- CTD[Central Touchscreen]
```

### 2.2 Component Diagram

```mermaid
graph LR
    subgraph "Sensor Layer"
        OS[Oxygen Sensors]
    end

    subgraph "Control Layer"
        ESP[ESP32 Microcontrollers]
        ZRP[Zone Raspberry Pi]
        ERlay[Ethernet Relays]
        TM[Timer Module]
    end

    subgraph "Management Layer"
        CRP[Central Raspberry Pi]
        DB[(Database)]
        WI[Web Interface]
        SCH[Scheduler]
        UM[User Management]
        ZM[Zone Management]
    end

    OS -->|Data Collection| ESP
    ESP -->|BLE Communication| ZRP
    ZRP -->|Control Commands| ERlay
    ZRP -->|Data Transmission| CRP
    TM -->|Time-based Control| ZRP
    CRP -->|Data Storage| DB
    CRP -->|User Interface| WI
    SCH -->|Schedule Management| CRP
    UM -->|Authentication & Authorization| CRP
    ZM -->|Zone Assignment & Control| CRP
```

### 2.3 Deployment Architecture

```mermaid
graph TD
    subgraph "Sensor-based Zone"
        OxySensor[Oxygen Sensor] --> ESP32
        ESP32 -->|BLE| ZoneRPi1[Zone Raspberry Pi]
        ZoneRPi1 -->|HTTP/API| EthRelay1[Ethernet Relay]
        EthRelay1 --> OxyUnit1[Oxygen Unit]
        ZoneRPi1 --- Display1[7-inch Touchscreen]
    end
    
    subgraph "Timer-based Zone"
        ZoneRPi2[Zone Raspberry Pi] -->|HTTP/API| EthRelay2[Ethernet Relay]
        EthRelay2 --> OxyUnit2[Oxygen Unit]
        ZoneRPi2 --- Display2[7-inch Touchscreen]
    end

    subgraph "Network Layer"
        ZoneRPi1 -->|Ethernet| Switch[Ethernet Switch]
        ZoneRPi2 -->|Ethernet| Switch
        Switch -->|Ethernet| CentralRPi[Central Raspberry Pi]
    end

    subgraph "Management Layer"
        CentralRPi --- CDisplay[7-inch Touchscreen]
        CentralRPi --- SQLite[(SQLite Database)]
        CentralRPi --- WebServer[Flask Web Server]
        CentralRPi --- Scheduler[Time Scheduler]
        CentralRPi --- UserManagement[User Authentication]
        CentralRPi --- ZoneManager[Zone Assignment]
    end
```

## 3. Low-Level Design (LLD)

### 3.1 Hardware Components

| Component | Specifications | Purpose |
|-----------|---------------|---------|
| ESP32 Microcontroller | Dual-core, BLE | Communication bridge |
| Oxygen Sensor | Electrochemical/Zirconia-based | Monitor oxygen levels |
| Raspberry Pi 4 | Quad-core, 4GB+ RAM | Zone control and display |
| 7" Touchscreen | 800x480 resolution | User interface |
| 16-channel Ethernet Relay | 12V/24V SPDT, 8 GPIOs | Control oxygen units |
| Ethernet Switch | Gigabit, managed | Network connectivity |
| Oxygen Unit | Commercial grade | Oxygen enrichment |

### 3.2 Software Components

#### Microcontroller Firmware (ESP32)
- **Language**: C/C++
- **Key Functions**:
  - Sensor data acquisition
  - Data preprocessing
  - BLE communication with Raspberry Pi
  - Low-level error handling

#### Zone Raspberry Pi Software
- **Language**: Python
- **Key Functions**:
  - BLE data reception (sensor-based zones)
  - Local python script collecting sensor data via BLE and sending to central via API request logic
  - ESP32 gpio pin control via BLE
  - Local logging and monitoring
  - Local python script to get MAC address and open chromium browser in kiosk mode
  - Central Raspberry Pi Software accessed through broswer and support 
    - UI rendering for touchscreen (including boost mode button)
    - Guest interface for zone activation/deactivation
    - Guest interface for boost mode control
    - Guest interface to control target altitude equivalency
    - Zone assignment interface for technician
    - Communication with central RPi
    - Ethernet relay control
    - Scheduled operation management

#### Central Raspberry Pi Software
- **Language**: Python
- **Frameworks**: Flask
- **Key Functions**:
  - Technician authentication system
  - Zone configuration (sensor-based or timer-based)
  - Zone assignment management
  - Time slot management
  - Boost mode duration configuration
  - System-wide monitoring
  - Database management
  - Web interface hosting
  - Alert and notification system
  - Centralized logging and reporting
  - Relay control logic
  - ESP32 gpio control logic
  - SMTP configuration and email sender
  - Scheduler for time-based operations
  - System dashboard for technicians
  - Zone maintenance lock functionality

### 3.3 Database Schema

```mermaid
erDiagram
    ZONE {
        int zone_id PK
        string zone_name
        string zone_relay
        string zone_type
        float target_altitude_equiv
        timestamp created_at
        timestamp updated_at
        boolean is_active
        boolean is_assigned
        boolean is_locked
        time operation_start
        time operation_end
    }
    
    DEVICE_REGISTRY {
        int zone_id FK
        string MAC
    }
    
    SENSOR {
        int sensor_id PK
        int zone_id FK
        string sensor_type
        string model
        timestamp last_calibrated
        boolean is_active
    }
    
    TIME_SLOT {
        int slot_id PK
        int zone_id FK
        time start_time
        time end_time
        boolean is_active
        int weekday_mask
    }
    
    OXYGEN_READING {
        int reading_id PK
        int sensor_id FK
        float oxygen_level
        float altitude_equiv
        timestamp reading_time
    }
    
    RELAY_STATUS {
        int relay_id PK
        int zone_id FK
        boolean is_on
        int channel_number
        timestamp last_toggled
        string trigger_source
    }
    
    SYSTEM_LOG {
        int log_id PK
        int zone_id FK
        string event_type
        string description
        int severity_level
        timestamp event_time
    }
    
    BOOST_MODE {
        int boost_id PK
        int zone_id FK
        int duration_minutes
        timestamp activated_at
        boolean is_active
    }
    
    GLOBAL_SETTINGS {
        int setting_id PK
        string setting_name
        string setting_value
        string setting_type
        string description
        timestamp updated_at
    }
    
    TECHNICIAN {
        int tech_id PK
        string username
        string password_hash
        timestamp last_login
        timestamp created_at
    }
    
    ZONE ||--o{ SENSOR : has
    ZONE ||--o{ TIME_SLOT : schedules
    SENSOR ||--o{ OXYGEN_READING : generates
    ZONE ||--o{ RELAY_STATUS : controls
    ZONE ||--o{ SYSTEM_LOG : produces
    ZONE ||--o{ BOOST_MODE : enables
```

### 3.4 API Design

#### Zone RPi API Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/sensor/data` | GET | Retrieve latest sensor data |
| `/api/relay/status` | GET | Get current relay status |
| `/api/relay/toggle` | POST | Toggle relay state |
| `/api/settings` | GET/POST | Retrieve/update zone settings |
| `/api/boost` | POST | Activate boost mode |
| `/api/boost/cancel` | POST | Cancel active boost mode |
| `/api/timeslots` | GET | Get configured time slots |
| `/api/zone/assign` | POST | Assign zone to a room |
| `/api/zone/unassign` | POST | Unassign zone from room |
| `/api/zone/activate` | POST | Activate zone operation |
| `/api/zone/deactivate` | POST | Deactivate zone operation |

#### Central RPi API Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/auth/login` | POST | Technician login |
| `/api/auth/logout` | POST | Technician logout |
| `/api/zones` | GET | List all zones |
| `/api/zones/{id}` | GET/PUT/DELETE | Get/update/delete specific zone |
| `/api/zones/{id}/history` | GET | Get historical data for zone |
| `/api/zones/{id}/timeslots` | GET/POST | Get/set time slots for zone |
| `/api/zones/{id}/operatinghours` | GET/POST | Get/set operating hours |
| `/api/zones/{id}/assign` | POST | Assign zone |
| `/api/zones/{id}/unassign` | POST | Unassign zone |
| `/api/zones/{id}/lock` | POST | Lock zone for maintenance |
| `/api/zones/{id}/unlock` | POST | Unlock zone from maintenance |
| `/api/zones/{id}/boost` | POST | Activate/deactivate boost mode |
| `/api/system/status` | GET | Get overall system status |
| `/api/system/dashboard` | GET | Get dashboard summary stats |
| `/api/system/logs` | GET | Retrieve system logs |
| `/api/settings/global` | GET/POST | Get/set global settings |
| `/api/settings/boost` | GET/POST | Get/set global boost duration |


### 3.5 Sensor Sampling Framework

#### Sampling Frequency Configuration

| Zone Type | Normal Sampling Rate | Alert Mode Sampling Rate |
|-----------|----------------------|--------------------------|
| Standard Guest Zones | Every 60 seconds | Every 10 seconds |
| High-Altitude Simulation Zones | Every 30 seconds | Every 5 seconds |
| Common Areas | Every 120 seconds | Every 30 seconds |


#### Data Processing Pipeline
1. **Raw Data Collection**:
   - ESP32 reads oxygen sensor values at configured intervals
   - Multiple readings taken in rapid succession (5 samples over 1 second)

2. **Signal Processing**:
   - Implementation of Kalman filtering to reduce sensor noise
   - Moving average (5-point) to smooth rapid fluctuations
   - Outlier detection and rejection (±3σ from running average)

3. **Data Transmission**:
   - Processed data sent to zone RPi via BLE
   - Transmission occurs immediately after processing
   - Batching mechanism for bandwidth optimization (up to 5 readings)

4. **Adaptive Sampling**:
   - Dynamic adjustment based on system conditions
   - Increased frequency when readings approach thresholds
   - Reduced frequency during stable periods to conserve power
   - Paused during non-operational hours

#### Data Retention Policies

| Data Type | Zone Storage | Central Storage | Archiving |
|-----------|--------------|-----------------|-----------|
| Raw readings | 24 hours | 7 days | Aggregated after 7 days |
| Processed readings | 7 days | 30 days | 1 year |
| Aggregated hourly averages | 30 days | 1 year | 5 years |
| System events | 7 days | 90 days | 1 year |


### 3.6 Logging and Monitoring System

#### Logging Levels

| Level | Description | Example | Storage Location |
|-------|-------------|---------|------------------|
| CRITICAL (1) | System failures requiring immediate attention | Oxygen unit failure, sensor disconnection | Zone RPi, Central RPi, Push notification |
| ERROR (2) | Operational errors affecting functionality | Communication failure, relay malfunction | Zone RPi, Central RPi |
| WARNING (3) | Conditions requiring attention but not immediate | Oxygen levels approaching thresholds, calibration needed | Zone RPi, Central RPi |
| INFO (4) | Normal operational events | System startup, configuration changes, relay toggling | Zone RPi, Central RPi |
| DEBUG (5) | Detailed information for troubleshooting | Sensor raw values, BLE packet details | Zone RPi only (configurable) |

## 4. Zone Operation Modes

### 4.1 Sensor-based Zone Operation

Sensor-based zones rely on oxygen sensors to determine when to activate the oxygen enrichment system:

1. **Core Operation:**
   - Oxygen sensors continuously monitor O₂ levels in the zone
   - System compares readings against the configured target altitude equivalency
   - Oxygen supply is activated when levels fall below the threshold
   - Supply is deactivated when levels exceed the threshold plus a configured hysteresis

2. **Operating Hours:**
   - Each sensor-based zone has configurable operation start/end times
   - During non-operational hours, the sensor monitoring continues but automated control is disabled
   - Manual overrides through boost mode remain available even during non-operational hours
   - Operating hours are configured by technicians during setup

3. **Scheduled Time Slots:**
   - In addition to sensor-based control, technician-configured time slots can be defined
   - During these slots, oxygen supply is activated regardless of sensor readings
   - This ensures scheduled oxygen enrichment at specific times
   - Useful for preemptive enrichment before peak usage hours

### 4.2 Timer-based Zone Operation

Timer-based zones operate exclusively on predefined schedules:

1. **Core Operation:**
   - No oxygen sensors are installed in these zones
   - Oxygen enrichment solely activated based on configured time slots
   - Multiple time slots can be defined for each day of the week
   - Time slots include start time, end time, and days of the week (using bitmask)

2. **Time Slot Configuration:**
   - Technicians configure time slots during zone setup
   - Interface allows multiple slots per day with minute-level granularity
   - Recurring patterns can be set (e.g., weekdays only, weekends only, specific days)
   - Slots can be temporarily disabled without deletion

3. **Overlap Handling:**
   - System handles overlapping time slots by merging them into continuous periods
   - Gap detection ensures optimal oxygen level maintenance
   - Configurable minimum gap between slots to prevent rapid cycling of equipment

### 4.3 Boost Mode Operation

Boost mode provides on-demand oxygen enrichment:

1. **User Activation:**
   - Guest-accessible boost button on zone touchscreen interface
   - Technician-accessible boost button on central interface
   - Configurable duration set by technician in global settings

2. **Duration Control:**
   - Technician configures global default boost duration
   - Can be overridden for specific zones if needed
   - Typical duration range: 10-60 minutes

3. **Operation Logic:**
   - When activated, relay immediately turns ON oxygen supply
   - Countdown timer displayed on zone interface
   - User can manually deactivate before timeout
   - System automatically deactivates after duration expires
   - Boost operation records stored in database for analysis

4. **Priority Handling:**
   - Boost mode takes priority over all other control mechanisms
   - When boost is active, sensor-based and timer-based controls are temporarily suspended
   - After boost concludes, zone returns to normal operation mode

## 5. Data Flow Diagrams

### 5.1 Sensor Data Flow

```mermaid
sequenceDiagram
    participant OS as Oxygen Sensor
    participant ESP as ESP32
    participant ZRP as Zone RPi
    participant CRP as Central RPi
    participant DB as Database

    OS->>ESP: Raw oxygen data
    ESP->>ESP: Process data
    ESP->>ESP: Calculate altitude equivalency
    ESP->>ZRP: Send processed data via BLE
    ZRP->>ZRP: Store temporary data
    ZRP->>ZRP: Check if within operating hours
    ZRP->>ZRP: Apply control logic if active
    ZRP->>CRP: Forward data via Ethernet
    CRP->>DB: Store in database
    CRP->>CRP: Update real-time monitoring
```

### 5.2 Control System Flow

```mermaid
sequenceDiagram
    participant ZRP as Zone RPi
    participant ER as Ethernet Relay
    participant OU as Oxygen Unit
    participant CRP as Central RPi
    
    Note over ZRP,OU: Local Control Loop
    ZRP->>ZRP: Check zone type (sensor/timer)
    ZRP->>ZRP: Check if zone is locked for maintenance
    ZRP->>ZRP: Check if zone is assigned and activated
    
    alt Zone Not Locked AND Assigned AND Activated
        alt Sensor-based Zone
            ZRP->>ZRP: Check if within operating hours
            ZRP->>ZRP: Check oxygen levels if active
            ZRP->>ZRP: Compare with thresholds
            
            alt Oxygen below threshold & within operating hours
                ZRP->>ER: Send ON command
                ER->>OU: Activate oxygen supply
            else Oxygen above threshold or outside operating hours
                ZRP->>ER: Send OFF command
                ER->>OU: Deactivate oxygen supply
            end
        else Timer-based Zone
            ZRP->>ZRP: Check if current time matches any time slot
            
            alt Within configured time slot
                ZRP->>ER: Send ON command
                ER->>OU: Activate oxygen supply
            else Outside any time slot
                ZRP->>ER: Send OFF command
                ER->>OU: Deactivate oxygen supply
            end
        end
    else Zone Locked OR Not Assigned OR Not Activated
        ZRP->>ER: Send OFF command
        ER->>OU: Deactivate oxygen supply
    end
    
    ZRP->>CRP: Report control action
    CRP->>CRP: Log action
    
    Note over CRP,ZRP: Central Override
    CRP->>ZRP: Send control override (if any)
    ZRP->>ER: Execute override command
```

### 5.3 User Interaction Flow

```mermaid
sequenceDiagram
    participant Guest
    participant Tech as Technician
    participant ZD as Zone Display
    participant ZRP as Zone RPi
    participant CD as Central Display
    participant CRP as Central RPi
    participant DB as Database
    
    alt Guest Zone Interaction
        Guest->>ZD: View zone status
        ZD->>ZRP: Request data
        ZRP->>ZD: Display data
        
        Guest->>ZD: Press boost button
        ZD->>ZRP: Activate boost mode
        ZRP->>ER: Turn ON oxygen supply
        ZRP->>CRP: Report boost activation
        CRP->>DB: Log boost event
        
        Guest->>ZD: Cancel boost
        ZD->>ZRP: Deactivate boost mode
        ZRP->>ER: Return to normal control
        
        Guest->>ZD: Activate/Deactivate zone
        ZD->>ZRP: Update zone status
        ZRP->>CRP: Sync changes
        CRP->>DB: Store updated settings
    else Technician Zone Setup
        Tech->>ZD: Login to zone setup
        ZD->>CRP: Authenticate technician
        CRP->>ZD: Authorization response
        
        Tech->>ZD: Assign zone
        ZD->>ZRP: Set zone assignment
        ZRP->>CRP: Update zone status
        CRP->>DB: Store assignment
    else Technician Central Management
        Tech->>CD: Login to technician interface
        CD->>CRP: Authenticate technician
        CRP->>CD: Display dashboard
        
        Tech->>CD: View system summary
        CD->>CRP: Request dashboard data
        CRP->>DB: Query statistics
        DB->>CRP: Return data
        CRP->>CD: Display system summary
        
        Tech->>CD: Modify global settings
        CD->>CRP: Update settings
        CRP->>DB: Store new settings
        CRP->>ZRP: Push settings to zones
        
        Tech->>CD: Manage zone configuration
        CD->>CRP: Update zone settings
        CRP->>DB: Store changes
        CRP->>ZRP: Push configuration to zone
    end
```

### 5.4 Time-based Control Flow

```mermaid
sequenceDiagram
    participant SCH as Scheduler
    participant CRP as Central RPi
    participant ZRP as Zone RPi
    participant ER as Ethernet Relay
    participant DB as Database
    
    Note over SCH,DB: Time Slot Processing
    SCH->>SCH: Periodic time check (every minute)
    SCH->>DB: Query upcoming time slots
    DB->>SCH: Return active slots
    
    loop For each zone with active time slot
        SCH->>CRP: Notify slot activation
        CRP->>ZRP: Check if zone is locked, assigned and activated
        
        alt Zone Not Locked AND Assigned AND Activated
            CRP->>ZRP: Send activation command
            ZRP->>ER: Turn ON oxygen supply
            ZRP->>CRP: Confirm activation
            CRP->>DB: Log event
        end
    end
    
    loop For each zone with ending time slot
        SCH->>CRP: Notify slot completion
        CRP->>ZRP: Check for other active conditions
        
        alt No other active conditions
            ZRP->>ER: Turn OFF oxygen supply
        else Other conditions active (e.g., sensor threshold, boost)
            ZRP->>ZRP: Maintain current state
        end
        
        ZRP->>CRP: Report status
        CRP->>DB: Log event
    end
    
    Note over SCH,DB: Boost Mode Timing
    SCH->>DB: Query active boost sessions
    DB->>SCH: Return expiring boosts
    
    loop For each expiring boost
        SCH->>CRP: Notify boost expiration
        CRP->>ZRP: Send expiration command
        ZRP->>ZRP: Check for other active conditions
        
        alt No other active conditions
            ZRP->>ER: Turn OFF oxygen supply
        else Other conditions active
            ZRP->>ZRP: Maintain current state
        end
        
        ZRP->>CRP: Report status
        CRP->>DB: Update boost record
    end
```

## 6. User Management System

### 6.1 Authentication System

The system implements a single-user authentication model for technicians only:

1. **Technician Authentication:**
   - Single technician account with password protection
   - Password creation/reset through terminal commands:
     - `flask create-superuser`
     - `flask update-superuser`
   - Session-based authentication using secure cookies
   - Automatic session timeout after period of inactivity

2. **Login Types:**
   - **Technician Setup Login:** Provides access to the central system management interface
   - **Zone Setup Login:** Provides access to zone assignment interface on individual zone displays

3. **Security Features:**
   - Password hashing using bcrypt
   - Rate limiting for login attempts
   - Session invalidation on logout
   - Activity logging for audit trail

### 6.2 Technician Interface

The technician interface provides comprehensive system management capabilities:

1. **Summary Dashboard:**
   - Total zones created vs. assigned
   - Breakdown of assigned zones (sensor-based vs. timer-based)
   - Currently active zones (activated by guests)
   - Zones with oxygen supply currently ON
   - Zones with active boost mode
   - Zones locked for maintenance
   - System health indicators

2. **Global Configuration:**
   - Company logo upload and display settings
   - Current altitude setting
   - Min/Max oxygen supply range configuration
   - Default boost mode duration
   - Zone-to-relay channel mapping
   - System-wide operating parameters

3. **Zone Management:**
   - Create/Update/Delete zones
   - Zone assignment status management
   - Zone activation/deactivation controls
   - Boost mode control for each zone
   - Maintenance lock functionality
   - Sensor-based zone monitoring (current altitude equivalency)
   - Historical data visualization

4. **Zone Control Panel:**
   - Consolidated view of all assigned zones
   - Quick activate/deactivate controls
   - Boost mode toggle buttons
   - Current status indicators
   - Real-time sensor readings for sensor-based zones

### 6.3 Guest Interface

The guest interface is limited to basic zone control functions:

1. **Zone Display Interface:**
   - Current oxygen level or altitude equivalency (sensor-based zones)
   - Zone activation/deactivation control
   - Target altitude equivalency setting (within allowed range)
   - Boost mode toggle button with countdown timer
   - Status indicators (maintenance lock, assignment status)

2. **Access Controls:**
   - Guests can only interact with assigned and unlocked zones
   - Settings are constrained within technician-defined limits
   - No authentication required for guest interface

### 6.4 Zone Assignment Process

1. **Initial Setup:**
   - Technician logs into Zone Setup on the zone's display
   - System presents list of available (unassigned and unlocked) zones
   - Technician selects appropriate zone for assignment
   - System confirms assignment and activates zone configuration

2. **Post-Assignment:**
   - Zone display switches to guest interface
   - Zone is marked as assigned in central database
   - Zone appears in assigned zone list on technician dashboard
   - Zone becomes available for guest control

3. **Unassignment Process:**
   - Technician can unassign zone through zone management interface
   - Unassignment deactivates the zone and disables guest control
   - Zone display reverts to unassigned state
   - Zone becomes available for reassignment

## 7. User Interface Design

### 7.1 Technician Dashboard

The central technician dashboard will feature:

1. **Summary Statistics Panel:**
   - Visual indicators for zone statistics (charts and counters)
   - Color-coded status indicators
   - System health monitoring gauges

2. **Quick Control Panel:**
   - Filterable list of all zones
   - Toggle switches for zone activation/deactivation
   - Boost mode controls
   - Maintenance lock controls

3. **Global Settings Panel:**
   - Form-based configuration interface
   - Logo preview and upload function
   - System parameters with validation
   - Save/reset controls

4. **Navigation Menu:**
   - Dashboard view
   - Zone management
   - System settings
   - Logs and reports
   - User management

### 7.2 Zone Setup Interface

The zone setup interface on individual displays will include:

1. **Authentication Screen:**
   - Username/password input for technician
   - Secure login process

2. **Zone Assignment Screen:**
   - List of available zones with details
   - Search/filter functionality
   - Assignment confirmation dialog
   - Cancel option to return to login

3. **Zone Configuration:**
   - Zone-specific settings after assignment
   - Operating parameters configuration
   - Time slot setup for timer-based zones
   - Sensor calibration options for sensor-based zones

### 7.3 Guest Control Interface

The zone control interface for guests will feature:

1. **Status Display:**
   - Current oxygen level (sensor-based zones)
   - Target altitude equivalency display
   - Zone status indicator (active/inactive)
   - Visual feedback for boost mode
   - Maintenance lock status indicator

2. **Control Elements:**
   - On/Off toggle switch for zone activation
   - Boost mode button with timer display
   - Target altitude adjustment controls (within allowed range)
   - Simple, intuitive interface suitable for guests

## 8. Processing and Management Plan

### 8.1 Processing Optimization

- Implement multi-threading for sensor data processing
- Optimize database queries with proper indexing
- Use in-memory caching for frequently accessed data
- Implement efficient data aggregation algorithms
- Schedule-based processing with priority queuing for timer functions
- Optimized zone status polling for dashboard updates

### 8.2 Data Management Strategy

- Implement data retention policies (raw data vs. aggregated data)
- Use time-based partitioning for sensor readings
- Implement data archiving for historical analysis
- Optimized storage for time slot configurations with efficient lookup
- Regular database maintenance and optimization tasks

### 8.3 Current Infrastructure

- Wired Ethernet for zone-to-central communication
- BLE for sensor-to-zone communication
- NTP synchronization for accurate time-based operations
- Local file storage for logs and configuration backup
- Secure authentication for technician access

## 9. References

1. STM32 Microcontroller Documentation
2. ESP32 Technical Reference Manual
3. Raspberry Pi 4 Datasheet
4. Web Ethernet Relay Technical Specifications
5. Flask Web Framework Documentation
6. SQLite Documentation
7. IoT System Architecture Best Practices
8. Time-based Control Systems Standards