# XRF Middleware Component Architecture

## Overview
This document maps all use cases from Master Backend Usecases.md to logical middleware components. The architecture follows a layered approach where components encapsulate related functionality and interact through well-defined interfaces.

---

## Component Hierarchy

### XRF API Manager (Top-Level Container)
All middleware components operate under the XRF API Manager, which provides the unified interface to the UI layer and coordinates cross-component operations.

---

## Core Middleware Components

### 1. Test Manager
**Responsibility**: Orchestrates all XRF measurement operations, test execution, and test sequencing.

**Use Cases**:
- Start/Stop Test
- Multi-Test Sequencing
- Repeat/Batch Tests
- Adaptive Time Testing
- Live Spectrum Display
- Workstation Interlock (Optional)

**Key Functions**:
- Test state machine management (idle → running → stopping → complete)
- Test timing control (min/max durations, beam sequencing)
- Multi-test and batch test orchestration
- Adaptive parameter optimization during measurement
- Real-time spectrum streaming coordination
- Hardware interlock enforcement

---

### 2. Safety Manager
**Responsibility**: Enforces all safety rules, interlocks, and emergency shutdown procedures.

**Use Cases**:
- Safety Triggers (deadman mode, trigger lock, two-handed operation)
- Radiation LED Safety Control
- Built-In Interlocks

**Key Functions**:
- Safety trigger mode enforcement
- Radiation LED synchronization with X-ray tube
- LED circuit integrity monitoring
- Emergency stop coordination
- Safety interlock validation (sample present, chamber door, workstation)
- Safety event logging and audit trail
- Cross-validation between LED state and tube state

---

### 3. Method Manager
**Responsibility**: Manages analysis methods, calibrations, and method-specific configurations.

**Use Cases**:
- Method Selection
- User Factors (Site Calibration)
- Method Display Preferences
- Element Display Order
- Pseudo-Element Formulas
- Compound Calculations (GeoChem Methods)
- RoHS Action Levels
- RoHS Classification Mode

**Key Functions**:
- Method library management and loading
- Method parameter configuration
- User factor/offset calculations
- Display preference management per method
- Pseudo-element formula evaluation
- Compound (oxide) calculation for Geochem
- RoHS-specific pass/fail threshold management
- Method validation and activation

---

### 4. Results Manager
**Responsibility**: Processes, stores, retrieves, and manages all test results and associated data.

**Use Cases**:
- Composition and Grade Identification
- Result Management (browse, delete)
- Scrap Type Classification (Recycling Mode)

**Key Functions**:
- Test result storage with timestamps
- Result browsing by date hierarchy (year/month/day)
- Result deletion with confirmation
- Result metadata management (notes, images, spectrum, GPS)
- Scrap classification result storage with value metrics
- Running totals and statistics for scrap sorting

---

### 5. Calc Manager
**Responsibility**: Interfaces with Calculation Engine (CalcEngine) for spectral analysis and chemistry calculations.

**Use Cases**:
- Composition and Grade Identification (chemistry calculation)
- Adaptive Time Testing (real-time spectrum analysis)
- Scrap Type Classification (composition analysis)

**Key Functions**:
- Spectrum processing coordination
- Elemental concentration calculation
- CalcEngine process management
- Real-time spectrum quality assessment
- Light element detection probability analysis
- Chemistry data formatting for UI/export

**Sub-Component**:
- **Calc Engine Process**: Separate process performing actual spectral deconvolution and chemistry calculations

---

### 6. Grade Library Manager
**Responsibility**: Manages grade libraries, grade matching algorithms, and confidence calculations.

**Use Cases**:
- Grade Library Management (library activation)
- Composition and Grade Identification (match number calculation)
- Scrap Type Classification (scrap-specific library matching)

**Key Functions**:
- Grade library loading and activation
- Match number calculation (0 = exact match)
- Multi-grade comparison and ranking
- Confidence score calculation
- Scrap-specific library hierarchical navigation
- Grade alert and message retrieval
- Value metric calculation for scrap types

---

### 7. Settings Manager
**Responsibility**: Centralized configuration management for all system settings.

**Use Cases**:
- Test Timing
- Custom Notes
- Display & Language Settings
- GPS Tagging
- Date/Time Sync

**Key Functions**:
- Settings persistence and retrieval
- Configuration validation and dependency checking
- Display settings (brightness, font size, rotation, language)
- GPS enable/disable and coordinate management
- Date/time synchronization (network or manual)
- Note template management
- Settings change notification to affected components

---

### 8. Export Manager
**Responsibility**: Handles all data export operations including configuration, formatting, and destination management.

**Use Cases**:
- Export Configuration
- Auto-Export
- Manual Export (Export Today)
- Browse/Filter & Select (export subset)
- Device Log Export

**Key Functions**:
- Export template management (content selection, format, filename conventions)
- Auto-export after test completion
- Manual export (today's results, selected results)
- Export destination configuration (SD, USB, network, cloud)
- Data formatting (CSV, PDF)
- Log export with category selection and filtering
- Date range filtering for exports
- Export status tracking and error handling

---

### 9. Network Manager
**Responsibility**: Manages all network connectivity including shares, cloud, and data synchronization.

**Use Cases**:
- Network Folders (CIFS/SMB mounting)
- Cloud Connectivity (Evident Cloud integration)
- Date/Time Sync (network time)

**Key Functions**:
- Network folder mounting/unmounting (CIFS/SMB)
- Network credential management
- Cloud service pairing and authentication (PIN-based)
- Cloud data synchronization
- Network time protocol (NTP) synchronization
- Network status monitoring
- Connection error handling and retry logic

---

### 10. Device Manager
**Responsibility**: Manages device hardware state, peripherals, and connectivity.

**Use Cases**:
- Bluetooth & Printing
- Camera and Imaging (Aiming Camera, Panoramic Camera, Quick Camera Controls)

**Key Functions**:
- Bluetooth radio control (on/off)
- Bluetooth device pairing (printer support)
- Print job management to paired printers
- Camera control (aiming camera, panoramic camera)
- IR collimator control
- Image capture and temporary storage
- Camera switching and view management
- Image association with test results

---

### 11. Diagnostics Manager
**Responsibility**: System diagnostics, health monitoring, and maintenance operations.

**Use Cases**:
- Hardware Diagnostics
- Device Log Export (diagnostics aspect)
- About Device
- Calibration Check

**Key Functions**:
- Hardware status monitoring (battery, system board, sensors)
- Diagnostic data collection and display
- Log file management (system, hardware, safety, error logs)
- Device information retrieval (model, serial, versions)
- Calibration verification using reference coin (316 SS)
- Temperature and voltage monitoring
- Error log analysis

---

### 12. Update Manager
**Responsibility**: Software and firmware update management.

**Use Cases**:
- Firmware and Software Updates (automatic and manual)

**Key Functions**:
- Automatic update checking (periodic, on login)
- Manual update check initiation
- Update server communication
- Version comparison and update detection
- Software update installation (UI, middleware, database)
- Firmware update installation (X-ray controller, DPP, drivers)
- Digital signature and checksum validation
- System backup before update
- Pre-update validation (storage, battery, no active scans)
- Update rollback on failure
- Update event logging
- Post-update verification

---

### 13. Session Manager
**Responsibility**: User session lifecycle management.

**Use Cases**:
- Session Login
- Session Logout

**Key Functions**:
- Session initiation and tracking
- Session state management
- Session termination and cleanup
- Session data persistence
- Welcome screen navigation
- Session timeout handling (if applicable)

---

### 14. Power Manager
**Responsibility**: Power state management and transitions.

**Use Cases**:
- Shutdown/Sleep Modes
- Wake from Sleep

**Key Functions**:
- Shutdown sequence orchestration
- Sleep mode entry with low-power state
- Wake trigger handling (touch, buttons, timers, network events)
- Component power-down coordination (X-ray tube, DPP, LED, display)
- Safety checks before shutdown/sleep (no active scan, unsaved data)
- System state save/restore
- Power state transitions
- Boot sequence coordination

---

### 15. Notification Handler
**Responsibility**: Event notification and UI alert management.

**Use Cases**:
- (Implicit across multiple components - handles notifications for updates, errors, safety events, etc.)

**Key Functions**:
- Event notification queue management
- UI notification banner display
- System tray notification coordination
- Priority-based notification routing
- Notification persistence for critical events
- User acknowledgment tracking

---

### 16. Dispatcher
**Responsibility**: API request routing and inter-component communication coordination.

**Use Cases**:
- (Infrastructure component - no direct use cases, but enables all API interactions)

**Key Functions**:
- API request parsing and routing
- Component interface orchestration
- Request validation
- Response aggregation
- Error propagation
- Event broadcasting to subscribers
- Asynchronous operation coordination

---

### 17. Common Utils
**Responsibility**: Shared utilities and helper functions used across components.

**Use Cases**:
- (Infrastructure component - supports all use cases with common functionality)

**Key Functions**:
- Data validation utilities
- String formatting and localization helpers
- Unit conversion utilities
- Timestamp and date formatting
- File I/O helpers
- Data serialization/deserialization
- Logging utilities
- Error code definitions
- Constants and configuration defaults

---

## Component Interaction Patterns

### Test Execution Flow:
```
UI → Dispatcher → Test Manager → Safety Manager (validation)
                                → Method Manager (parameters)
                                → HW API (DPP, Tube, LED)
                                → Calc Manager → CalcEngine
                                → Results Manager (storage)
                                → Grade Library Manager (matching)
                                → Notification Handler (UI updates)
```

### Export Flow:
```
UI → Dispatcher → Export Manager → Results Manager (data retrieval)
                                 → Network Manager (destination validation)
                                 → Device Manager (USB/SD check)
                                 → Settings Manager (export template)
                                 → Database (data formatting)
```

### Safety Event Flow:
```
HW API (LED failure) → Safety Manager → Test Manager (emergency stop)
                                      → Notification Handler (alert)
                                      → Diagnostics Manager (logging)
```

### Update Flow:
```
Timer/User → Update Manager → Network Manager (server connection)
                            → Diagnostics Manager (pre-update checks)
                            → Power Manager (restart coordination)
                            → Notification Handler (user prompts)
```

---

## Cross-Cutting Concerns

### Database Integration:
All managers that persist data interact with the Database layer:
- Results Manager (test results, chemistry, images, spectrum)
- Method Manager (method configurations, user factors)
- Grade Library Manager (grade definitions, libraries)
- Settings Manager (all configuration settings)
- Diagnostics Manager (logs, diagnostic history)
- Export Manager (export templates, history)
- Network Manager (network credentials, mounted shares)
- Session Manager (session state)

### Safety Integration:
All managers that trigger X-ray operations must coordinate with Safety Manager:
- Test Manager (primary coordinator)
- Method Manager (parameter validation)
- Diagnostics Manager (calibration check)
- Power Manager (shutdown coordination)

### Hardware API Integration:
Components that directly interface with Hardware API:
- Test Manager (DPP, Tube control)
- Safety Manager (Radiation LED, interlocks)
- Diagnostics Manager (hardware status queries)
- Device Manager (camera hardware)
- Power Manager (hardware power control)

---

## Notes

1. **Component Independence**: Each component should have well-defined interfaces and minimal coupling to other components. Communication occurs through the Dispatcher or direct API calls with clear contracts.

2. **State Management**: Components maintain their own internal state but coordinate state changes through events and notifications.

3. **Error Handling**: Each component is responsible for its own error handling but propagates critical errors through standardized error codes via Dispatcher.

4. **Logging**: All components log significant events through Common Utils logging interface for audit trail and diagnostics.

5. **Extensibility**: New functionality (e.g., new application modes like Mining, Jewelry) should be accommodated by extending existing managers rather than creating new ones where possible.

6. **Thread Safety**: Components that handle real-time data (Test Manager, Calc Manager) must implement thread-safe operations for concurrent access.