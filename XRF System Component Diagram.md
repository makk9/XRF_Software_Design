# XRF Analyzer System - Component Diagram

## High-Level Architecture Overview

Based on the comprehensive use case analysis, the XRF analyzer system follows a layered architecture with clear separation of concerns and well-defined component interactions.

## Component Diagram

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                                  XRF ANALYZER SYSTEM                            │
└─────────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────────┐
│                                  UI LAYER                                       │
├─────────────────────────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────┐  │
│  │   Live      │  │  Settings   │  │   Grade     │  │   System    │  │  User   │  │
│  │ Measurement │  │    Menu     │  │  Matching   │  │   Health    │  │  Auth   │  │
│  │   Screen    │  │ Interface   │  │ Interface   │  │ Dashboard   │  │ Screen  │  │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘  └─────────┘  │
│         │                 │                 │                 │           │       │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────┐  │
│  │  Spectrum   │  │ Data Sync/  │  │  Alloy      │  │ Calibration │  │ Test    │  │
│  │   Chart     │  │   Export    │  │ Mismatch    │  │   Wizard    │  │ Mode    │  │
│  │  Display    │  │ Interface   │  │  Display    │  │             │  │ Control │  │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘  └─────────┘  │
└─────────────────────────────────────────────────────────────────────────────────┘
                                        │
                              ┌─────────▼─────────┐
                              │   UI Events &     │
                              │  State Manager    │
                              └─────────┬─────────┘
                                        │
┌─────────────────────────────────────────────────────────────────────────────────┐
│                              MIDDLEWARE API LAYER                               │
├─────────────────────────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────┐  │
│  │    Scan     │  │   Grade     │  │   System    │  │    User     │  │  Data   │  │
│  │ Controller  │  │   Match     │  │  Settings   │  │ Session     │  │ Export  │  │
│  │             │  │  Engine     │  │   Manager   │  │  Manager    │  │ Manager │  │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘  └─────────┘  │
│         │                 │                 │                 │           │       │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────┐  │
│  │   Safety    │  │  Test Mode  │  │ Calibration │  │   Health    │  │ Config  │  │
│  │ Interlock   │  │   Manager   │  │   Manager   │  │  Monitor    │  │  XRF    │  │
│  │ Coordinator │  │             │  │             │  │             │  │ Manager │  │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘  └─────────┘  │
│                                        │                                          │
│  ┌─────────────────────────────────────▼──────────────────────────────────────┐  │
│  │                      MIDDLEWARE CORE ENGINE                                │  │
│  │  • State Machine Management    • Data Processing Pipeline                 │  │
│  │  • API Orchestration          • Real-time Data Streaming                 │  │
│  │  • Business Logic Validation  • Error Handling & Recovery                │  │
│  │  • Hardware Abstraction       • Audit Trail Management                   │  │
│  └────────────────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────────────┘
                                        │
┌─────────────────────────────────────────────────────────────────────────────────┐
│                               CALCULATION ENGINE                                │
├─────────────────────────────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────────────────────────────────────────┐ │
│  │                        CHEMISTRY CALCULATION ENGINE                         │ │
│  │  • Peak Identification Algorithms    • Elemental Concentration Calculations │ │
│  │  • Matrix Correction Processing      • Uncertainty Estimation              │ │
│  │  • Machine Learning Classification   • Adaptive Testing Algorithms         │ │
│  │  • Spectral Analysis & Processing    • Statistical Confidence Calculation  │ │
│  └─────────────────────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────────────────┘
                                        │
┌─────────────────────────────────────────────────────────────────────────────────┐
│                               HARDWARE API LAYER                                │
├─────────────────────────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────┐  │
│  │   X-ray     │  │     DPP     │  │   Safety    │  │  Filter     │  │ System  │  │
│  │    Tube     │  │ Controller  │  │    LED      │  │   Wheel     │  │ Sensors │  │
│  │ Controller  │  │             │  │ Controller  │  │ Controller  │  │         │  │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘  └─────────┘  │
│         │                 │                 │                 │           │       │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────┐  │
│  │   Camera    │  │  Storage    │  │  Network    │  │   Power     │  │  GPS    │  │
│  │ Controller  │  │ Controller  │  │ Controller  │  │ Management  │  │ Module  │  │
│  │             │  │ (USB/SD)    │  │             │  │             │  │         │  │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘  └─────────┘  │
│                                        │                                          │
│  ┌─────────────────────────────────────▼──────────────────────────────────────┐  │
│  │                      HARDWARE ABSTRACTION LAYER                           │  │
│  │  • Device Driver Management       • Hardware State Monitoring             │  │
│  │  • Safety Interlock Integration   • Firmware Update Management            │  │
│  │  • Hardware Constraint Validation • Device Communication Protocols        │  │
│  │  • Real-time Hardware Monitoring  • Emergency Shutdown Coordination       │  │
│  └────────────────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────────────┘
                                        │
┌─────────────────────────────────────────────────────────────────────────────────┐
│                               DATABASE LAYER                                    │
├─────────────────────────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────┐  │
│  │    Scan     │  │ Chemistry   │  │  Spectrum   │  │  Metadata   │  │  User   │  │
│  │  Sessions   │  │   Results   │  │    Data     │  │   Tables    │  │  Data   │  │
│  │   Table     │  │   Table     │  │   Table     │  │             │  │ Table   │  │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘  └─────────┘  │
│         │                 │                 │                 │           │       │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────┐  │
│  │   Grade     │  │ Calibration │  │   System    │  │    Audit    │  │ Config  │  │
│  │ Libraries   │  │   History   │  │    Logs     │  │    Trail    │  │ Store   │  │
│  │   Table     │  │   Table     │  │   Table     │  │   Table     │  │ Table   │  │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘  └─────────┘  │
└─────────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────────┐
│                              EXTERNAL SYSTEMS                                   │
├─────────────────────────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────┐  │
│  │   Cloud     │  │   Update    │  │  External   │  │   Network   │  │   USB   │  │
│  │  Storage    │  │   Server    │  │  Database   │  │   Printer   │  │ Device  │  │
│  │ Services    │  │             │  │             │  │             │  │         │  │
│  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘  └─────────┘  │
└─────────────────────────────────────────────────────────────────────────────────┘
```

## Key Architectural Patterns

### 1. **Layered Architecture**
- **UI Layer**: User interface components and display logic
- **Middleware API Layer**: Business logic, orchestration, and state management
- **Calculation Engine**: Specialized chemistry and analysis algorithms
- **Hardware API Layer**: Hardware abstraction and device control
- **Database Layer**: Data persistence and query management

### 2. **Component Communication Patterns**

**Synchronous Communication:**
- UI → Middleware API (REST-like calls)
- Middleware API → Hardware API (direct hardware control)
- Middleware API → Calculation Engine (chemistry analysis)
- Middleware API → Database (data persistence)

**Asynchronous Communication:**
- Hardware API → Middleware API (live data streaming)
- Middleware API → UI (real-time updates)
- Hardware API → Middleware API (safety events)

### 3. **Core System Components**

**UI Layer Components:**
- Live measurement interfaces (spectrum, chemistry display)
- Settings and configuration interfaces
- Grade matching and analysis displays
- System health and diagnostic dashboards
- User authentication and session management

**Middleware API Components:**
- **Scan Controller**: Manages test lifecycle and state machine
- **Grade Match Engine**: Handles grade identification and confidence
- **System Settings Manager**: Configuration and XRF settings
- **Safety Interlock Coordinator**: Safety system integration
- **Test Mode Manager**: PMI/reCycling mode switching
- **Health Monitor**: System health and diagnostics
- **Data Export Manager**: Export and sync operations

**Hardware API Components:**
- **X-ray Tube Controller**: Tube voltage, current, safety control
- **DPP Controller**: Digital pulse processor and spectrum collection
- **Safety LED Controller**: Radiation warning indicator
- **Filter Wheel Controller**: Automated filter positioning
- **System Sensors**: Temperature, humidity, power monitoring
- **Storage Controllers**: USB, SD card access
- **Camera Controller**: Image capture for metadata

**Database Components:**
- Scan session data with complete audit trails
- Chemistry results with uncertainties and confidence
- Spectrum data storage and retrieval
- Metadata tables (images, GPS, notes)
- User management and authentication
- System configuration and settings
- Grade libraries and calibration data

### 4. **Key Integration Points**

**Safety Integration:**
- Hardware safety interlocks operate independently
- Middleware coordinates safety state across system
- UI displays safety warnings and status
- Database logs all safety events

**Real-time Data Flow:**
- Hardware → Middleware → UI (spectrum/chemistry streaming)
- Buffering and processing in middleware layer
- Asynchronous updates to prevent UI blocking

**Configuration Management:**
- Hierarchical settings (System → Mode → Component → User)
- Hardware constraint validation
- Cross-component dependency resolution
- Settings persistence and versioning

**State Management:**
- Centralized state machine in middleware
- State synchronization across all components
- Event-driven state transitions
- Audit trail for all state changes

## Design Principles Observed

1. **Separation of Concerns**: Clear separation between UI, business logic, and hardware
2. **Hardware Abstraction**: Middleware insulated from hardware specifics
3. **Safety First**: Independent safety systems with coordinated responses
4. **Data Integrity**: Comprehensive audit trails and transaction management
5. **Extensibility**: Mode-based architecture supports future applications
6. **Real-time Performance**: Streaming data architecture for live updates
7. **Configuration Management**: Flexible, hierarchical settings system
8. **User Experience**: Responsive UI with async operations and progress feedback

This architecture provides a robust, scalable foundation for the XRF analyzer system with clear interfaces, comprehensive safety integration, and support for multiple application modes while maintaining regulatory compliance and audit trail requirements.