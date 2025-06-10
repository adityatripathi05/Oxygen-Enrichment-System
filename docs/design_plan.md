# Oxygen Enrichment Control System Design_v0.1
## Architecture Plan

## Table of Contents
- [Oxygen Enrichment Control System Design\_v0.1](#oxygen-enrichment-control-system-design_v01)
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
  - [4. Data Flow Diagrams](#4-data-flow-diagrams)
    - [4.1 Sensor Data Flow](#41-sensor-data-flow)
    - [4.2 Control System Flow](#42-control-system-flow)
    - [4.3 User Interaction Flow](#43-user-interaction-flow)
  - [5. Processing and Management Plan](#5-processing-and-management-plan)
      - [Processing Optimization](#processing-optimization)
      - [Data Management Strategy](#data-management-strategy)
      - [Current Infrastructure](#current-infrastructure)
  - [6. References](#6-references)

## 1. System Overview

The Oxygen Enrichment Control System is designed for vacation guest houses to monitor and manage oxygen levels in different zones. The system uses oxygen sensors connected to ESP32 microcontrollers that communicate via Bluetooth with zone-specific Raspberry Pi units. Each zone has an oxygen supply unit controlled through Web Ethernet relays. The central Raspberry Pi collects data from all zones and provides system-wide monitoring and control.

**Key Features:**
- Real-time oxygen level monitoring
- Automated oxygen enrichment based on predefined thresholds
- Altitude equivalency calculation
- Zone-specific monitoring and control
- Centralized management interface
- Local touchscreen displays in each zone

## 2. High-Level Design (HLD)

### 2.1 System Architecture

```mermaid
graph TD
    subgraph "Zone 1"
        OS1[Oxygen Sensor] --> ESP1[ESP32 Microcontroller]
        ESP1 -->|BLE| RPI1[Zone Raspberry Pi]
        RPI1 -->|Ethernet| ER1[Ethernet Relay]
        ER1 --> OU1[Oxygen Unit]
        RPI1 -->|Ethernet| SW[Ethernet Switch]
    end

    subgraph "Zone 2"
        OS2[Oxygen Sensor] --> ESP2[ESP32 Microcontroller]
        ESP2 -->|BLE| RPI2[Zone Raspberry Pi]
        RPI2 -->|Ethernet| ER2[Ethernet Relay]
        ER2 --> OU2[Oxygen Unit]
        RPI2 -->|Ethernet| SW
    end

    subgraph "Zone N"
        OSN[Oxygen Sensor] --> ESPN[ESP32 Microcontroller]
        ESPN -->|BLE| RPIN[Zone Raspberry Pi]
        RPIN -->|Ethernet| ERN[Ethernet Relay]
        ERN --> OUN[Oxygen Unit]
        RPIN -->|Ethernet| SW
    end

    SW -->|Ethernet| CRPI[Central Raspberry Pi]
    CRPI --> DB[(SQLite Database)]
    CRPI --> WI[Web Interface]
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
    end

    subgraph "Management Layer"
        CRP[Central Raspberry Pi]
        DB[(Database)]
        WI[Web Interface]
    end

    OS -->|Data Collection| ESP
    ESP -->|BLE Communication| ZRP
    ZRP -->|Control Commands| ERlay
    ZRP -->|Data Transmission| CRP
    CRP -->|Data Storage| DB
    CRP -->|User Interface| WI
```

### 2.3 Deployment Architecture

```mermaid
graph TD
    subgraph "Physical Zone"
        OxySensor[Oxygen Sensor] --> ESP32
        ESP32 -->|BLE| ZoneRPi[Zone Raspberry Pi]
        ZoneRPi -->|HTTP/API| EthRelay[Ethernet Relay]
        EthRelay --> OxyUnit[Oxygen Unit]
        ZoneRPi --- Display[7-inch Touchscreen]
    end

    subgraph "Network Layer"
        ZoneRPi -->|Ethernet| Switch[Ethernet Switch]
        Switch -->|Ethernet| CentralRPi[Central Raspberry Pi]
    end

    subgraph "Management Layer"
        CentralRPi --- CDisplay[7-inch Touchscreen]
        CentralRPi --- SQLite[(SQLite Database)]
        CentralRPi --- WebServer[Flask Web Server]
    end
```

## 3. Low-Level Design (LLD)

### 3.1 Hardware Components

| Component | Specifications | Purpose |
|-----------|---------------|---------|
| STM32 Microcontroller | ARM Cortex-M4, Low-power | Process sensor data |
| ESP32 | Dual-core, Wi-Fi/BLE | Communication bridge |
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
- **Frameworks**: Flask
- **Key Functions**:
  - BLE data reception
  - Local python script collecting sensor data via. BLE and sending to central via. API request logic
  - Local flask app to get MAC address 
  - UI rendering for touchscreen
  - Communication with central RPi
  - Ethernet relay control
  - ESP32 gpio pin control via. BLE
  - Local logging and monitoring

#### Central Raspberry Pi Software
- **Language**: Python
- **Frameworks**: Flask
- **Key Functions**:
  - Zone configuration
  - System-wide monitoring
  - Database management
  - Web interface hosting
  - Alert and notification system
  - Centralized logging and reporting
  - Relay control logic
  - ESP32 gpio control logic
  - SMTP configuration and email sender

### 3.3 Database Schema

```mermaid
erDiagram
    ZONE {
        int zone_id PK
        string zone_name
		string zone_relay
        float target_altitude_equiv
        timestamp created_at
        timestamp updated_at
		boolean is_active
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
    }
    
    SYSTEM_LOG {
        int log_id PK
        int zone_id FK
        string event_type
        string description
        int severity_level
        timestamp event_time
    }
    
    ZONE ||--o{ SENSOR : has
    SENSOR ||--o{ OXYGEN_READING : generates
    ZONE ||--o{ RELAY_STATUS : controls
    ZONE ||--o{ SYSTEM_LOG : produces
```

### 3.4 API Design

#### Zone RPi API Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/sensor/data` | GET | Retrieve latest sensor data |
| `/api/relay/status` | GET | Get current relay status |
| `/api/relay/toggle` | POST | Toggle relay state |
| `/api/settings` | GET/POST | Retrieve/update zone settings |

#### Central RPi API Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/zones` | GET | List all zones |
| `/api/zones/{id}` | GET | Get specific zone details |
| `/api/zones/{id}/history` | GET | Get historical data for zone |
| `/api/system/status` | GET | Get overall system status |
| `/api/system/logs` | GET | Retrieve system logs |

## 4. Data Flow Diagrams

### 4.1 Sensor Data Flow

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
    ZRP->>ZRP: Apply control logic
    ZRP->>CRP: Forward data via Ethernet
    CRP->>DB: Store in database
    CRP->>CRP: Update real-time monitoring
```

### 4.2 Control System Flow

```mermaid
sequenceDiagram
    participant ZRP as Zone RPi
    participant ER as Ethernet Relay
    participant OU as Oxygen Unit
    participant CRP as Central RPi
    
    Note over ZRP,OU: Local Control Loop
    ZRP->>ZRP: Check oxygen levels
    ZRP->>ZRP: Compare with thresholds
    
    alt Oxygen below threshold
        ZRP->>ER: Send ON command
        ER->>OU: Activate oxygen supply
    else Oxygen above threshold
        ZRP->>ER: Send OFF command
        ER->>OU: Deactivate oxygen supply
    end
    
    ZRP->>CRP: Report control action
    CRP->>CRP: Log action
    
    Note over CRP,ZRP: Central Override
    CRP->>ZRP: Send control override (if any)
    ZRP->>ER: Execute override command
```

### 4.3 User Interaction Flow

```mermaid
sequenceDiagram
    participant User
    participant ZD as Zone Display
    participant ZRP as Zone RPi
    participant CD as Central Display
    participant CRP as Central RPi
    participant DB as Database
    
    alt Zone-level Interaction
        User->>ZD: View zone status
        ZD->>ZRP: Request data
        ZRP->>ZD: Display data
        
        User->>ZD: Adjust zone settings
        ZD->>ZRP: Update settings
        ZRP->>CRP: Sync changes
        CRP->>DB: Store updated settings
    else Central-level Interaction
        User->>CD: View system dashboard
        CD->>CRP: Request system data
        CRP->>DB: Query database
        DB->>CRP: Return data
        CRP->>CD: Display system status
        
        User->>CD: Modify system settings
        CD->>CRP: Update settings
        CRP->>DB: Store new settings
        CRP->>ZRP: Push settings to zones
    end
```

## 5. Processing and Management Plan

#### Processing Optimization

- Implement multi-threading for sensor data processing
- Optimize database queries with proper indexing
- Use in-memory caching for frequently accessed data
- Implement efficient data aggregation algorithms

#### Data Management Strategy

- Implement data retention policies (raw data vs. aggregated data)
- Use time-based partitioning for sensor readings
- Implement data archiving for historical analysis

#### Current Infrastructure

- Wired Ethernet for zone-to-central communication
- BLE for sensor-to-zone communication

## 6. References

1. STM32 Microcontroller Documentation
2. ESP32 Technical Reference Manual
3. Raspberry Pi 4 Datasheet
4. Web Ethernet Relay Technical Specifications
5. Flask Web Framework Documentation
6. PostgreSQL Documentation
7. IoT System Architecture Best Practices
