# XRF Analyzer System - API Sequence Diagrams

## Use Case 1: Start Scan

### Overview
User initiates X-ray measurement by pressing start button. System activates hardware components and begins data collection while ensuring safety protocols.

### Sequence Flow

```
Actor: User
UI Layer: User Interface
MW API: Middleware API  
HW API: Hardware API
DB: Database
Safety: Safety System
```

**Step-by-Step API Flow:**

1. **User Action**
   ```
   User -> UI: Clicks start/play button
   ```

2. **UI Request to Middleware**
   ```
   UI -> MW API: POST /api/scan/start (async)
   ```

3. **State Management & Validation**
   ```
   MW API: Check current system state (must be "Idle")
   MW API: Generate scan session ID
   MW API: Update state to "Starting"
   ```

4. **Safety Pre-checks**
   ```
   MW API -> HW API: checkSafety() - safety interlocks 
   HW API -> MW API: Return safety status (OK/BLOCKED)
   ```

5. **Hardware Activation**
   ```
   MW API -> HW API: startXRayTube() (sync)
   HW API: Activate X-ray tube (voltage, current)
   HW API -> MW API: Return tube status (SUCCESS/FAILED)
   
   MW API -> HW API: startDPP() (sync)  
   HW API: Initialize Digital Pulse Processor
   HW API -> MW API: Return DPP status (SUCCESS/FAILED)
   ```

6. **Safety LED Activation**
   ```
   MW API -> HW API: enableRadiationLED() (sync)
   HW API: Turn on radiation warning LED
   HW API -> MW API: Return LED status (SUCCESS/FAILED)
   ```

7. **State Update & Response**
   ```
   MW API: Update state to "Active" 
   MW API -> DB: Log scan start event (session_id, timestamp, user)
   MW API -> UI: Return scan session details - voltage, current, etc. (async response)
   ```

8. **UI Update**
   ```
   UI: Update interface to show "Scan Active" status
   UI: Display scan session information
   ```

### State Transitions
- **Initial State**: Idle
- **Transition State**: Starting (during hardware activation)
- **Final State**: Active (ready for data collection)

### API Sequence Diagram

```
User    UI       MW API            HW API    DB
 |       |         |                 |       |
 |------>|         |                 |       |  1. Click start button
 |       |-------->|                 |       |  2. POST /api/scan/start (async)
 |       |         | [verify safety] |       |  3. MW API internal safety checks
 |       |         |---------------->|       |  4. startXRayTube() (sync)
 |       |         |<----------------|       |  5. Return tube status (SUCCESS)
 |       |         |---------------->|       |  6. startDPP() (sync)
 |       |         |<----------------|       |  7. Return DPP status (SUCCESS)
 |       |         |---------------->|       |  8. enableRadiationLED() (sync)
 |       |         |<----------------|       |  9. Return LED status (SUCCESS)
 |       |         |------------------------>| 10. Log scan start event
 |       |<--------|                 |       | 11. Return scan session details (async)
 |       |         |                 |       |
```

### Success Criteria
- X-ray tube activated successfully
- DPP initialized and ready for data collection  
- Radiation LED activated (safety requirement)
- System state updated to "Active"
- Scan session created and logged
---

## Use Case 2: View Spectrum (Live)

### Overview
Real-time display of X-ray spectrum data during active scan. System continuously streams spectrum data from hardware through middleware to UI for live visualization.

### Sequence Flow

```
Actor: System (Continuous Process)
UI Layer: User Interface
MW API: Middleware API  
HW API: Hardware API
```

**Step-by-Step API Flow:**

1. **Hardware Data Collection** (Continuous during scan)
   ```
   HW API: DPP collects X-ray spectrum data
   HW API: Package spectrum data with metadata (timestamp, count_rate, etc.)
   ```

2. **Data Streaming to Middleware**
   ```
   HW API -> MW API: streamSpectrumData(spectrum_data, metadata) (async)
   ```

3. **Middleware Processing**
   ```
   MW API: Parse raw spectrum data
   MW API: Apply calibration corrections *needed?
   MW API: Format data for UI consumption
   MW API: Update spectrum buffer/cache
   ```

4. **Data Push to UI**
   ```
   MW API -> UI: pushSpectrumUpdate(formatted_spectrum) (async)
   ```

5. **UI Visualization Update**
   ```
   UI: Receive spectrum data
   UI: Build/Update real-time spectrum chart w/ charting library
   UI: Add spectrum to carousel spectrum view
   UI: Refresh display with new data points
   ```

6. **Continuous Loop**
   ```
   Process repeats every [X ms] while scan is active
   ```


### API Sequence Diagram

```
System   HW API    MW API            UI
 |        |         |                |
 |        | [continuous data collection] |  1. DPP collects spectrum data
 |        |-------->|                |  2. streamSpectrumData() (async)
 |        |         | [parse & cal]  |  3. MW API processes data
 |        |         |--------------->|  4. pushSpectrumUpdate() (async)
 |        |         |                | [update chart] 5. UI refreshes display
 |        |         |                |
 |        | [loop continues while scan active]
 |        |         |                |
```

### Success Criteria
- Smooth real-time spectrum visualization
- Minimal latency between hardware and display
- Proper data calibration and formatting
- No UI blocking during updates

---

## Use Case 3: View Chemistry (Live)

### Overview
Real-time calculation and display of elemental chemistry analysis during active scan. System processes spectrum data through calculation engine to provide live chemistry breakdown to user interface.

### Sequence Flow

```
Actor: System (Continuous Process)
UI Layer: User Interface
MW API: Middleware API  
Calc Engine: Chemistry Calculation Engine
HW API: Hardware API
```

**Step-by-Step API Flow:**

1. **Hardware Data Collection** (Continuous during scan)
   ```
   HW API: DPP collects X-ray spectrum data
   HW API: Package spectrum data with metadata (timestamp, count_rate, etc.)
   ```

2. **Data Streaming to Middleware**
   ```
   HW API -> MW API: streamSpectrumData(spectrum_data, metadata) (async)
   ```

3. **Middleware Processing & Calc Engine Request**
   ```
   MW API: Parse raw spectrum data
   MW API: Apply calibration corrections
   MW API -> Calc Engine: analyzeChemistry(calibrated_spectrum) (sync)
   ```

4. **Chemistry Calculation**
   ```
   Calc Engine: Apply peak identification algorithms
   Calc Engine: Calculate elemental concentrations
   Calc Engine: Apply matrix corrections
   Calc Engine: Generate uncertainty estimates
   ```

5. **Chemistry Results Return**
   ```
   Calc Engine -> MW API: Return chemistry_results (element_list, concentrations, uncertainties)
   ```

6. **Formatted Data Push to UI**
   ```
   MW API: Format chemistry data for UI display
   MW API -> UI: pushChemistryUpdate(formatted_chemistry) (async)
   ```

7. **UI Chemistry Table Update**
   ```
   UI: Receive chemistry data
   UI: Update real-time chemistry table
   UI: Display element concentrations with uncertainties
   UI: Highlight significant changes from previous update
   ```

8. **Continuous Loop**
   ```
   Process repeats every [X ms] while scan is active
   ```

### API Sequence Diagram

```
System   HW API    MW API      Calc Engine    UI
 |        |         |             |           |
 |        | [continuous data collection]      |  1. DPP collects spectrum data
 |        |-------->|             |           |  2. streamSpectrumData() (async)
 |        |         | [parse & cal]           |  3. MW API processes data
 |        |         |------------>|           |  4. analyzeChemistry() (sync)
 |        |         |             | [calc]    |  5. Calculate concentrations
 |        |         |<------------|           |  6. Return chemistry_results
 |        |         | [format]    |           |  7. MW API formats data
 |        |         |------------------------>|  8. pushChemistryUpdate() (async)
 |        |         |             |           | [update table] 9. UI refreshes
 |        |         |             |           |
 |        | [loop continues while scan active]
 |        |         |             |           |
```

### Success Criteria
- Real-time chemistry table updates during scan
- Accurate elemental concentration calculations
- Proper uncertainty estimates displayed
- Smooth UI updates without blocking

---

## Use Case 4: Stop Scan

### Overview
User terminates active X-ray measurement by pressing stop button. System safely shuts down hardware components and updates system state while ensuring proper safety protocols.

### Sequence Flow

```
Actor: User
UI Layer: User Interface
MW API: Middleware API  
HW API: Hardware API
DB: Database
```

**Step-by-Step API Flow:**

1. **User Action**
   ```
   User -> UI: Clicks stop button
   ```

2. **UI Request to Middleware**
   ```
   UI -> MW API: POST /api/scan/stop (async)
   ```

3. **State Management & Validation**
   ```
   MW API: Check current system state (must be "Active" or "Scanning")
   MW API: Update state to "Stopping"
   ```

4. **Hardware Shutdown Sequence**
   ```
   MW API -> HW API: stopXRayTube() (sync)
   HW API: Safely shut down X-ray tube
   HW API -> MW API: Return tube stop status (SUCCESS/FAILED)
   
   MW API -> HW API: stopDPP() (sync)
   HW API: Stop Digital Pulse Processor data collection
   HW API -> MW API: Return DPP stop status (SUCCESS/FAILED)
   ```

5. **Safety LED Deactivation**
   ```
   MW API -> HW API: disableRadiationLED() (sync)
   HW API: Turn off radiation warning LED
   HW API -> MW API: Return LED status (SUCCESS/FAILED)
   ```

6. **Final State Update & Logging**
   ```
   MW API: Update state to "Idle"
   MW API -> DB: Log scan stop event (session_id, timestamp, duration)
   MW API -> UI: Return scan stop confirmation (async response)
   ```

7. **UI Update**
   ```
   UI: Update interface to show "Idle" status
   UI: Stop live data updates (spectrum, chemistry)
   UI: Display scan completed status
   ```

### API Sequence Diagram

```
User    UI       MW API            HW API    DB
 |       |         |                 |       |
 |------>|         |                 |       |  1. Click stop button
 |       |-------->|                 |       |  2. POST /api/scan/stop (async)
 |       |         | [state check]   |       |  3. MW API validates current state
 |       |         |---------------->|       |  4. stopXRayTube() (sync)
 |       |         |<----------------|       |  5. Return tube stop status
 |       |         |---------------->|       |  6. stopDPP() (sync)
 |       |         |<----------------|       |  7. Return DPP stop status
 |       |         |---------------->|       |  8. disableRadiationLED() (sync)
 |       |         |<----------------|       |  9. Return LED status
 |       |         |------------------------->| 10. Log scan stop event
 |       |<--------|                 |       | 11. Return stop confirmation (async)
 |       | [update UI to idle]       |       | 12. UI shows idle status
 |       |         |                 |       |
```

### State Transitions
- **Initial State**: Active/Scanning
- **Transition State**: Stopping (during hardware shutdown)
- **Final State**: Idle

### Key Design Decisions
- **Async UI**: UI doesn't block during hardware shutdown
- **Sync Hardware**: MW API waits for each component shutdown confirmation
- **Safety Integration**: LED deactivation is mandatory safety requirement
- **Orderly Shutdown**: Tube stops before DPP, LED turns off last
- **State Validation**: Only allow stop if system is currently active
- **Audit Trail**: All scan stops logged with duration tracking

### Success Criteria
- X-ray tube safely shut down
- DPP data collection stopped
- Radiation LED deactivated (safety requirement)
- System state updated to "Idle"
- Scan completion logged to database
- UI returns to idle state with live updates stopped

---

## Use Case 5: Save All Result

### Overview
Automatic saving of all scan results to database after test completion. System packages and persists spectrum data, chemistry analysis, metadata, and visual chart data for audit trails and future reference.

### Sequence Flow

```
Actor: System (Automatic Trigger)
UI Layer: User Interface
MW API: Middleware API  
DB: Database
```

**Step-by-Step API Flow:**

1. **Auto-Save Trigger** (After scan completion)
   ```
   MW API: Detect scan completion event
   MW API: Initiate auto-save process
   ```

2. **Data Collection & Packaging**
   ```
   MW API: Gather current spectrum data from buffer
   MW API: Collect latest chemistry analysis results
   MW API: Package scan metadata (session_id, duration, settings)
   MW API: Generate chart/visualization data
   MW API: Compile complete result dataset
   ```

3. **Database Transaction**
   ```
   MW API -> DB: saveCompleteResults(result_package) (sync)
   DB: Begin transaction
   DB: Insert spectrum data into spectrum_table
   DB: Insert chemistry data into chemistry_table
   DB: Insert metadata into scan_sessions_table
   DB: Insert chart data into visualizations_table
   DB: Commit transaction
   DB -> MW API: Return save status (SUCCESS/FAILED)
   ```

4. **UI Notification**
   ```
   MW API -> UI: notifySaveComplete(save_status) (async)
   UI: Display saving completion icon
   UI: Show save confirmation to user
   UI: Update scan status to "Saved"
   UI: Add saved spectrum to carousel
   ```

5. **Audit Logging**
   ```
   MW API -> DB: logSaveEvent(session_id, timestamp, data_size)
   ```

### API Sequence Diagram

```
System   MW API            DB                UI
 |        |                 |                 |
 | [scan complete trigger]  |                 |  1. Auto-save triggered
 |        | [collect data]  |                 |  2. MW API gathers all results
 |        |---------------->|                 |  3. saveCompleteResults() (sync)
 |        |                 | [transaction]   |  4. DB saves all data tables
 |        |<----------------|                 |  5. Return save status
 |        |--------------------------------->|  6. notifySaveComplete() (async)
 |        |                 |                 | [show icon] 7. UI shows save confirmation
 |        |---------------->|                 |  8. logSaveEvent() audit
 |        |                 |                 |
```

### Data Components Saved
- **Spectrum Data**: Raw and calibrated spectrum arrays, energy calibration
- **Chemistry Results**: Element concentrations, uncertainties, analysis parameters
- **Scan Metadata**: Session ID, timestamp, duration, operator, instrument settings
- **Visualization Data**: Chart configurations, display parameters, annotations
- **Audit Information**: Save timestamp, data integrity checksums, version info

### Key Design Decisions
- **Automatic Trigger**: Saves without user intervention after scan completion
- **Sync Database**: MW API waits for database confirmation before proceeding
- **Transactional**: All data saved atomically or rollback on failure
- **Comprehensive**: Includes all data types for complete audit trail
- **UI Feedback**: Visual confirmation of successful save operation
- **Audit Compliance**: Additional logging for regulatory requirements

### Success Criteria
- All scan data successfully persisted to database
- Complete result package saved atomically
- UI displays save confirmation to user
- Audit trail entry created for compliance
- Data integrity maintained throughout save process

---

## Use Case 6: Attach Metadata (Image, Notes)

### Overview
User adds contextual metadata to current test including photographs and text notes. GPS location is automatically captured based on system settings. System stores this supplementary data in dedicated metadata tables linked to scan sessions for comprehensive documentation and audit trail.

### Sequence Flow

```
Actor: User
UI Layer: User Interface
MW API: Middleware API  
HW API: Hardware API (Camera)
DB: Database
```

**Step-by-Step API Flow:**

### Sub-Flow A: Capture Image

1. **User Image Request**
   ```
   User -> UI: Presses camera button
   ```

2. **Camera Activation**
   ```
   UI -> MW API: POST /api/metadata/capture-image (async)
   MW API -> HW API: activateCamera() (sync)
   HW API: Initialize camera hardware
   HW API: Capture image
   HW API -> MW API: Return image_data (base64/binary)
   ```

3. **Image Storage**
   ```
   MW API: Associate image with current scan session
   MW API -> DB: saveImageMetadata(session_id, image_data, timestamp)
   MW API -> UI: Return image capture confirmation
   UI: Display image thumbnail confirmation
   ```

### Sub-Flow B: Add Text Notes

4. **User Notes Request**
   ```
   User -> UI: Presses Notes button
   UI: Opens text input dialog
   User -> UI: Enters text and saves
   ```

5. **Notes Storage**
   ```
   UI -> MW API: POST /api/metadata/save-notes (notes_text) (async)
   MW API: Associate notes with current scan session
   MW API -> DB: saveNotesMetadata(session_id, notes_text, timestamp)
   MW API -> UI: Return notes save confirmation
   UI: Close notes dialog, show confirmation
   ```

### Sub-Flow C: Automatic GPS Location (During Save Process)

6. **GPS Auto-Capture** (Triggered during scan save if GPS setting enabled)
   ```
   MW API: Check GPS settings (enabled/disabled)
   MW API -> HW API: getCurrentLocation() (sync) [if enabled]
   HW API: Access GPS hardware
   HW API: Get current coordinates, accuracy, timestamp
   HW API -> MW API: Return location_data (lat, lon, accuracy, timestamp)
   MW API -> DB: saveLocationMetadata(session_id, location_data)
   ```

### API Sequence Diagram

```
User    UI       MW API    HW API    DB
 |       |         |         |       |
 |------>|         |         |       |  1. Press camera button
 |       |-------->|         |       |  2. POST /api/metadata/capture-image
 |       |         |-------->|       |  3. activateCamera() (sync)
 |       |         |<--------|       |  4. Return image_data
 |       |         |---------------->|  5. saveImageMetadata()
 |       |<--------|         |       |  6. Image confirmation
 |       |         |         |       |
 |------>|         |         |       |  7. Press Notes button
 |       | [notes dialog]    |       |  8. User enters text
 |       |-------->|         |       |  9. POST /api/metadata/save-notes
 |       |         |---------------->| 10. saveNotesMetadata()
 |       |<--------|         |       | 11. Notes confirmation
 |       |         |         |       |
 |       |         | [auto GPS check]|       | 12. GPS auto-capture (during save)
 |       |         |-------->|       | 13. getCurrentLocation() [if enabled]
 |       |         |<--------|       | 14. Return location_data
 |       |         |---------------->| 15. saveLocationMetadata()
 |       |         |         |       |
```

### Database Storage Architecture
**Separate Linked Metadata Tables:**
- **scan_sessions**: Main scan data (spectrum, chemistry, timestamps)
- **image_metadata**: Photos linked via session_id (binary/base64, format, resolution)
- **location_metadata**: GPS data linked via session_id (lat, lon, accuracy, altitude)
- **notes_metadata**: Text notes linked via session_id (text, operator_id, timestamps)

**Benefits of Separate Tables:**
- Clean data separation and specialized indexing
- Flexible metadata queries independent of scan data
- Efficient storage for large binary image data
- Easy metadata reporting and audit trails

### Metadata Types
- **Image Data**: Photo binary/base64, timestamp, image format, resolution
- **GPS Location**: Latitude, longitude, accuracy radius, GPS timestamp, altitude (auto-captured)
- **Text Notes**: User-entered text, character limit, timestamp, operator ID
- **Session Association**: All metadata linked to current scan session ID

### Key Design Decisions
- **Session-Based Linking**: All metadata associated with active scan via session_id
- **Automatic GPS**: Location captured during save process based on system settings
- **User-Initiated Metadata**: Image and notes captured on-demand by user
- **Separate Database Tables**: Dedicated metadata tables linked to scan sessions
- **Hardware Integration**: Direct camera access through HW API
- **User Confirmation**: Visual feedback for successful metadata capture
- **Async UI**: Non-blocking operations for smooth user experience

### Success Criteria
- Image successfully captured and stored in dedicated metadata table
- GPS coordinates automatically retrieved during save (if enabled)
- Text notes properly associated with current test session
- All metadata linked to scan session for complete audit trail
- Metadata available for inclusion in final comprehensive test results

---

## Use Case 7: Safety Interlock

### Overview
Critical safety system that monitors trigger release and immediately stops X-ray emission if safety conditions are violated. Hardware-driven safety interlock provides immediate emergency shutdown with automatic UI notification and system state management.

### Sequence Flow

```
Actor: Hardware (Safety Trigger)
Mechanical: Physical safety trigger
HW API: Hardware API
MW API: Middleware API
UI Layer: User Interface
DB: Database
```

**Step-by-Step API Flow:**
*Potentially add a safety condition - checks object is on nose on device or depth 

### Normal Operation (Trigger Held)
1. **Trigger Monitor** (Continuous during scan)
   ```
   Mechanical: Safety trigger pressed and held
   HW API: Monitor trigger state continuously
   HW API: Maintain normal operation while trigger held
   ```

### Emergency Interlock Triggered

2. **Trigger Release Detection**
   ```
   Mechanical: User releases safety trigger
   HW API: Detect trigger release immediately
   HW API: Initiate emergency shutdown sequence
   ```

3. **Immediate Hardware Shutdown**
   ```
   HW API: Stop X-ray tube immediately (hardware interlock)
   HW API: Stop DPP data collection
   HW API: Disable radiation LED automatically
   HW API: Set hardware to safe state
   ```

4. **Emergency Notification to Middleware**
   ```
   HW API -> MW API: emergencyStop(trigger_release, timestamp) (async)
   ```

5. **Middleware Safety Response**
   ```
   MW API: Update system state to "Stop"
   MW API: Stop all active scan processes
   MW API: Prepare emergency stop notification
   ```

6. **UI Emergency Display**
   ```
   MW API -> UI: displayEmergencyStop(emergency_message) (async)
   UI: Display emergency stop message on live screen
   UI: Show safety warning overlay
   UI: Disable all scan controls
   UI: Display trigger re-engagement instructions
   ```

7. **Safety Event Logging**
   ```
   MW API -> DB: logSafetyEvent(session_id, trigger_release, timestamp)
   ```

8. **System Recovery** (After trigger re-engagement)
   ```
   * Question: Start button, Trigger flow
   Mechanical: User re-engages safety trigger
   HW API: Detect trigger engagement
   HW API -> MW API: triggerReengaged(timestamp)
   MW API: Update system state to "Idle"
   MW API -> UI: clearEmergencyStop()
   UI: Remove emergency overlay, enable controls
   ```

### API Sequence Diagram

```
Mechanical  HW API    MW API            UI       DB
    |        |         |                |        |
    |        | [continuous monitoring]  |        |  1. Monitor trigger state
    | [release] |      |                |        |  2. Trigger released
    |------->|         |                |        |  3. HW detects release
    |        | [immediate shutdown]     |        |  4. Stop tube/DPP/LED
    |        |-------->|                |        |  5. emergencyStop() (async)
    |        |         | [state update] |        |  6. MW API emergency response
    |        |         |--------------->|        |  7. displayEmergencyStop()
    |        |         |                | [show] |  8. UI shows emergency message
    |        |         |------------------------>|  9. logSafetyEvent()
    |        |         |                |        |
    | [re-engage] |    |                |        | 10. User re-engages trigger
    |------->|         |                |        | 11. HW detects engagement
    |        |-------->|                |        | 12. triggerReengaged()
    |        |         |--------------->|        | 13. clearEmergencyStop()
    |        |         |                | [clear]| 14. UI clears emergency display
    |        |         |                |        |
```

### Safety Requirements (CRITICAL)
- **Hardware Interlock**: Tube shutdown must be immediate and hardware-driven
- **Fail-Safe Design**: System defaults to safe state on any failure
- **Continuous Monitoring**: Trigger state monitored at hardware level
- **Immediate Response**: No software delays in emergency shutdown
- **User Feedback**: Clear emergency messaging and recovery instructions
- **Audit Trail**: All safety events logged for regulatory compliance

### Key Design Decisions
- **Hardware-Driven**: Safety interlock operates at hardware level, not software
- **Immediate Shutdown**: X-ray tube stops instantly on trigger release
- **Async Notification**: Safety events notified to software after hardware action
- **State Management**: MW API coordinates system state during safety events
- **Recovery Process**: Clear procedure for system restoration after safety event
- **Comprehensive Logging**: All safety events recorded for audit purposes

### Safety Event Types
- **Trigger Release**: Primary safety interlock activation
- **Hardware Fault**: Equipment malfunction triggering safety response
- **System Override**: Manual emergency stop activation
- **Recovery Events**: Trigger re-engagement and system restoration

### Success Criteria
- Immediate X-ray tube shutdown on trigger release
- Hardware safety interlock operates independently of software
- Clear emergency messaging displayed to user
- System state properly managed during safety events
- Complete audit trail of all safety-related events
- Proper system recovery procedure after safety event resolution

---

## Use Case 8: Radiation LED Control

### Overview
Critical safety system that provides visual radiation warning through physical LED synchronized with X-ray tube operation. System maintains test state machine coordination with mandatory LED control, circuit monitoring, and comprehensive safety feedback to users.

### Sequence Flow

```
Actor: System (State Machine Driven)
MW API: Middleware API (State Controller)
HW API: Hardware API
UI Layer: User Interface
DB: Database
```

**Step-by-Step API Flow:**

### State Transition: Idle → Active (LED ON)

1. **Test State Change**
   ```
   MW API: State machine transition (idle → preparing → active)
   MW API: Prepare LED activation command
   ```

2. **LED Activation Command**
   ```
   MW API -> HW API: enableRadiationLED() (sync)
   ```

3. **Hardware LED Control**
   ```
   HW API: Activate physical radiation LED
   HW API: Monitor LED circuit integrity
   HW API: Verify LED operational status
   HW API: Synchronize with X-ray tube activation
   ```

4. **LED Status Confirmation**
   ```
   HW API -> MW API: Return LED_status (SUCCESS/FAILED/FAULT)
   ```

5. **UI Radiation Warning Display**
   ```
   MW API -> UI: updateRadiationStatus(LED_ON, test_active) (async)
   UI: Display radiation warning icon/indicator
   UI: Show test status as "ACTIVE"
   UI: Display radiation safety warnings
   UI: Update screen with active emission indicators
   ```

6. **Safety Event Logging**
   ```
   MW API -> DB: logRadiationEvent(LED_ON, timestamp, session_id)
   ```

### Continuous Monitoring (During Active State)

7. **LED Circuit Monitoring** (Continuous)
   ```
   HW API: Monitor LED circuit integrity continuously
   HW API: Verify LED/X-ray tube synchronization
   HW API: Detect any LED faults or discrepancies
   ```

8. **Fault Detection & Reporting**
   ```
   HW API -> MW API: reportLEDFault(fault_type, timestamp) [if fault detected]
   MW API: Trigger emergency safety response if LED fault
   MW API -> UI: displayLEDFaultWarning(fault_details)
   ```

### State Transition: Active → Idle (LED OFF)

9. **Test Completion**
   ```
   MW API: State machine transition (active → complete → idle)
   MW API: Prepare LED deactivation command
   ```

10. **LED Deactivation Command**
    ```
    MW API -> HW API: disableRadiationLED() (sync)
    ```

11. **Hardware LED Shutdown**
    ```
    HW API: Deactivate physical radiation LED
    HW API: Verify LED OFF status
    HW API: Confirm X-ray tube synchronization
    ```

12. **LED Status Confirmation**
    ```
    HW API -> MW API: Return LED_status (OFF_SUCCESS/FAILED)
    ```

13. **UI Safety Status Update**
    ```
    MW API -> UI: updateRadiationStatus(LED_OFF, test_idle) (async)
    UI: Remove radiation warning indicators
    UI: Show test status as "IDLE"
    UI: Clear radiation safety warnings
    UI: Update to safe/idle state display
    ```

14. **Safety Event Logging**
    ```
    MW API -> DB: logRadiationEvent(LED_OFF, timestamp, session_id)
    ```

### API Sequence Diagram

```
System   MW API    HW API            UI       DB
 |        |         |                |        |
 | [state: idle->active] |           |        |  1. State machine transition
 |        |-------->|                |        |  2. enableRadiationLED() (sync)
 |        |         | [LED ON + monitor]      |  3. Activate LED & monitor
 |        |<--------|                |        |  4. Return LED_status (SUCCESS)
 |        |------------------------>|        |  5. updateRadiationStatus(LED_ON)
 |        |         |                | [show] |  6. UI shows radiation warnings
 |        |-------------------------->        |  7. logRadiationEvent(LED_ON)
 |        |         |                |        |
 |        |         | [continuous monitoring] |  8. Monitor LED circuit
 |        |         |                |        |
 | [state: active->idle] |           |        |  9. State machine transition
 |        |-------->|                |        | 10. disableRadiationLED() (sync)
 |        |         | [LED OFF + verify]      | 11. Deactivate LED & verify
 |        |<--------|                |        | 12. Return LED_status (OFF_SUCCESS)
 |        |------------------------>|        | 13. updateRadiationStatus(LED_OFF)
 |        |         |                | [clear]| 14. UI clears radiation warnings
 |        |-------------------------->        | 15. logRadiationEvent(LED_OFF)
 |        |         |                |        |
```

### Test State Machine Integration
- **Idle**: LED OFF, no radiation warnings, system ready
- **Preparing**: LED OFF, system preparing for activation
- **Active**: LED ON (mandatory), radiation warnings displayed, X-ray emission active
- **Complete**: LED OFF, completing scan, transitioning to idle

### LED Safety Requirements (CRITICAL)
- **Mandatory Activation**: LED must be ON during any X-ray emission
- **Hardware Synchronization**: LED state synchronized with X-ray tube
- **Fault Monitoring**: Continuous LED circuit integrity monitoring
- **Emergency Response**: LED failure triggers immediate safety response
- **User Visibility**: Clear visual indication of radiation hazard state
- **Audit Compliance**: All LED operations logged for regulatory requirements

### Key Design Decisions
- **State Machine Driven**: LED control tied to system state transitions
- **Synchronous Commands**: MW API waits for LED status confirmation
- **Continuous Monitoring**: Hardware monitors LED circuit integrity during operation
- **Fault Detection**: Immediate fault reporting and emergency response
- **User Safety**: Comprehensive UI warnings during radiation periods
- **Regulatory Logging**: Complete audit trail of all LED operations

### LED Status Types
- **SUCCESS**: LED activated/deactivated successfully
- **FAILED**: LED command failed to execute
- **FAULT**: LED circuit fault detected during operation
- **SYNC_ERROR**: LED/X-ray tube synchronization failure

### Success Criteria
- LED automatically activates/deactivates with test state changes
- Hardware LED synchronizes perfectly with X-ray tube operation
- UI provides clear radiation warning indicators during active periods
- LED circuit faults detected and reported immediately
- Complete audit trail of all LED control operations
- Emergency safety response triggered on LED failures

---

## Use Case 9: ChangeTestMode (PMI/reCycling Support)

### Overview
Critical system configuration that switches between application modes (PMI and reCycling) with distinct hardware and software settings. Each mode optimizes the XRF analyzer for specific applications with mode-specific tube parameters, filter positions, software features, and calibrations.

### Sequence Flow

```
Actor: User
UI Layer: User Interface
MW API: Middleware API  
HW API: Hardware API
DB: Database
```

**Step-by-Step API Flow:**

### Mode Selection Process

1. **User Mode Selection**
   ```
   User -> UI: Selects mode (PMI or reCycling) from mode selection interface
   ```

2. **Safety and State Validation**
   ```
   UI -> MW API: POST /api/mode/change-request (new_mode) (async)
   MW API: Check current system state (must not be "Active" - no X-ray emission)
   MW API: Validate mode switch is permitted
   MW API: Display confirmation dialog if active test in progress
   ```

3. **Mode Configuration Retrieval**
   ```
   MW API: Load mode-specific configuration profile from memory
   MW API: Retrieve mode-specific HW settings (kV, μA, filter, detector)
   MW API: Prepare mode-specific SW feature configuration
   ```

4. **Hardware Configuration Application**
   ```
   MW API -> HW API: configureMode(mode_settings) (sync)
   HW API: Set tube voltage for selected mode (e.g., 40kV PMI vs 50kV reCycling)
   HW API: Set tube current for selected mode (e.g., 200μA PMI vs 100μA reCycling)
   HW API: Position appropriate filter for mode (e.g., Filter 1 vs Filter 3)
   HW API: Configure detector settings for mode requirements
   HW API: Validate all hardware parameters set correctly
   HW API -> MW API: Return hardware_config_status (SUCCESS/FAILED)
   ```

5. **Software Feature Configuration**
   ```
   MW API: Enable/disable mode-specific software features
   MW API: PMI Mode - Enable grade matching, library access, confidence calculations
   MW API: reCycling Mode - Enable scrap sorting, fast analysis, contamination detection
   MW API: Load mode-specific calibration algorithms
   MW API: Configure mode-specific data processing parameters
   ```

6. **Mode State Update and UI Configuration**
   ```
   MW API: Update current test mode state
   MW API -> UI: updateModeConfiguration(mode_details, features, status) (async)
   UI: Update main screen to show current active mode prominently
   UI: Configure mode-specific UI screens and templates
   UI: Display mode-specific feature availability
   UI: Show mode-specific calibration status and readiness indicators
   UI: Update notes templates for selected mode
   ```

7. **Database Mode Logging**
   ```
   MW API -> DB: logModeChange(new_mode, timestamp, user_id, hardware_settings)
   ```

8. **Calibration Status Verification**
   ```
   MW API: Verify mode-specific calibration status
   MW API -> UI: updateCalibrationStatus(mode_calibration_readiness)
   UI: Display calibration readiness indicators for new mode
   ```

### API Sequence Diagram

```
User    UI       MW API            HW API    DB
 |       |         |                 |       |
 |------>|         |                 |       |  1. Select mode (PMI/reCycling)
 |       |-------->|                 |       |  2. POST /api/mode/change-request
 |       |         | [safety check]  |       |  3. Validate system state (not active)
 |       |         | [load config]   |       |  4. Retrieve mode configuration
 |       |         |---------------->|       |  5. configureMode(mode_settings) (sync)
 |       |         |                 | [HW]  |  6. Apply tube, filter, detector settings
 |       |         |<----------------|       |  7. Return hardware_config_status
 |       |         | [SW features]   |       |  8. Configure software features
 |       |<--------|                 |       |  9. updateModeConfiguration()
 |       | [UI update]               |       | 10. Update UI for new mode
 |       |         |------------------------->| 11. logModeChange()
 |       |         | [cal status]    |       | 12. Verify calibration readiness
 |       |<--------|                 |       | 13. updateCalibrationStatus()
 |       | [show ready]              |       | 14. Display mode readiness
 |       |         |                 |       |
```

### Mode-Specific Configurations

**PMI (Positive Material Identification) Mode:**
- **Hardware**: Higher voltage (40-50kV), moderate current (200μA), specific filter
- **Software**: Grade matching enabled, library access, confidence calculations, alloy verification
- **UI**: Grade match results, confidence levels, pass/fail indicators
- **Calibration**: Alloy-specific calibrations, grade libraries

**reCycling Mode:**
- **Hardware**: Optimized voltage (30-40kV), variable current (100-300μA), different filter
- **Software**: Fast analysis, scrap sorting, contamination detection, throughput optimization
- **UI**: Scrap classification, contamination warnings, sorting recommendations
- **Calibration**: Scrap-specific calibrations, contamination thresholds

### Safety Requirements (CRITICAL)
- **No Mode Switch During Emission**: Mode changes prohibited during active X-ray emission
- **Configuration Validation**: Hardware settings validated before activation
- **Calibration Verification**: Mode-specific calibrations checked before operation
- **User Confirmation**: Confirmation required when switching during active tests
- **Audit Trail**: All mode changes logged for regulatory compliance

### Key Design Decisions
- **Safety First**: Mode switching blocked during X-ray emission
- **Sync Hardware Config**: MW API waits for hardware configuration confirmation
- **Mode State Management**: MW API maintains current mode state throughout system
- **Feature Toggling**: Software features enabled/disabled based on mode
- **UI Adaptation**: Interface updates to reflect mode-specific capabilities
- **Extensible Design**: Architecture supports future modes (mining, jewelry, etc.)

### Future Mode Extensibility
- **Mining Mode**: Optimized for geological samples, different element focus
- **Jewelry Mode**: Precious metals analysis, different calibrations
- **Custom Modes**: User-configurable modes for specific applications

### Success Criteria
- Mode successfully changed with proper hardware configuration
- Software features correctly enabled/disabled for selected mode
- UI properly updated to reflect new mode capabilities
- Mode-specific calibrations verified and ready
- All mode changes logged for audit compliance
- System ready for operation in new mode configuration

---

## Use Case 10: Export Data

### Overview
User-initiated data export system that allows selection of test data by date range, configuration of export format/template, and generation of downloadable files. System packages comprehensive test data including spectrum, chemistry, metadata, and images based on user-selected features and templates.

### Sequence Flow

```
Actor: User
UI Layer: User Interface
MW API: Middleware API  
DB: Database
```

**Step-by-Step API Flow:**

### Data Selection Process

1. **Calendar View Request**
   ```
   User -> UI: Navigate to export/data management section
   UI -> MW API: GET /api/export/calendar-data (async)
   MW API -> DB: getTestDataByDate(date_range) 
   DB -> MW API: Return test_summary_list (dates, session_ids, modes, status)
   MW API -> UI: Return calendar_data (test_dates, summaries)
   UI: Display calendar view with available test data
   ```

2. **Data Selection**
   ```
   User -> UI: Selects specific dates/tests in calendar view
   UI: Highlight selected test data entries
   UI: Display selection summary (count, date range, total size)
   ```

3. **Format/Template Selection**
   ```
   User -> UI: Selects export format (PDF, Excel, CSV, XML)
   User -> UI: Selects template (Standard, Detailed, Custom, Regulatory)
   UI: Display template preview and included data fields
   UI: Show format-specific options (charts, images, metadata inclusion)
   ```

### Export Processing

4. **Export Request**
   ```
   User -> UI: Clicks export button
   UI -> MW API: POST /api/export/generate (selection_criteria, format, template) (async)
   ```

5. **Data Packaging Configuration**
   ```
   MW API: Parse selection criteria and template requirements
   MW API: Determine data components to include based on current feature settings
   MW API: Check user permissions for data export
   MW API: Generate export session ID for tracking
   ```

6. **Database Data Retrieval**
   ```
   MW API -> DB: getExportData(session_ids, include_spectrum, include_chemistry, include_metadata)
   DB: Query scan_sessions table for selected date range
   DB: Join with chemistry_table, spectrum_table, metadata tables
   DB: Retrieve image_metadata, location_metadata, notes_metadata
   DB -> MW API: Return comprehensive_data_package
   ```

7. **Data Processing and Formatting**
   ```
   MW API: Apply selected template formatting
   MW API: Generate charts/visualizations if required by template
   MW API: Format chemistry data according to template requirements
   MW API: Include/exclude spectrum data based on template settings
   MW API: Process metadata (images, GPS, notes) for inclusion
   MW API: Apply data filtering based on current feature settings
   ```

8. **File Generation**
   ```
   MW API: Generate file in selected format (PDF/Excel/CSV/XML)
   MW API: Apply template styling and layout
   MW API: Include regulatory compliance headers if required
   MW API: Generate file checksums for integrity verification
   MW API: Store export file temporarily for download
   ```

9. **Export Completion**
   ```
   MW API -> UI: exportComplete(download_url, file_info) (async)
   UI: Display export completion notification
   UI: Provide download link with file details
   UI: Show export summary (file size, records included, format)
   ```

10. **Download and Cleanup**
    ```
    User -> UI: Clicks download link
    UI: Initiate file download
    MW API: Log export event for audit trail
    MW API -> DB: logExportEvent(export_id, user_id, timestamp, data_scope)
    MW API: Schedule temporary file cleanup
    ```

### API Sequence Diagram

```
User    UI       MW API            DB
 |       |         |                |
 |------>|         |                |  1. Navigate to export section
 |       |-------->|                |  2. GET /api/export/calendar-data
 |       |         |--------------->|  3. getTestDataByDate()
 |       |         |<---------------|  4. Return test_summary_list
 |       |<--------|                |  5. Return calendar_data
 |       | [show calendar]          |  6. Display calendar view
 |------>|         |                |  7. Select dates/tests
 |       | [show selection]         |  8. Display selection summary
 |------>|         |                |  9. Select format/template
 |       | [show preview]           | 10. Display template preview
 |------>|         |                | 11. Click export button
 |       |-------->|                | 12. POST /api/export/generate
 |       |         | [config export]| 13. Configure data packaging
 |       |         |--------------->| 14. getExportData()
 |       |         |<---------------| 15. Return comprehensive_data_package
 |       |         | [process data] | 16. Format and generate file
 |       |<--------|                | 17. exportComplete(download_url)
 |       | [show ready]             | 18. Display download ready
 |------>|         |                | 19. Click download
 |       | [download file]          | 20. Initiate file download
 |       |         |--------------->| 21. logExportEvent()
 |       |         |                |
```

### Export Format Options
- **PDF**: Professional reports with charts, images, regulatory headers
- **Excel**: Spreadsheet format with multiple tabs (chemistry, spectrum, metadata)
- **CSV**: Simple data format for external analysis tools
- **XML**: Structured data format for system integration

### Template Types
- **Standard**: Basic chemistry results, test conditions, timestamps
- **Detailed**: Includes spectrum data, uncertainty values, calibration info
- **Custom**: User-configurable fields and layout options
- **Regulatory**: Compliance-focused with audit trails and certifications

### Data Inclusion Based on Features
- **Chemistry Data**: Element concentrations, uncertainties, detection limits
- **Spectrum Data**: Raw and processed spectrum arrays (optional)
- **Metadata**: Images, GPS location, notes, operator information
- **Test Conditions**: Mode, calibration, hardware settings, timestamps
- **Quality Data**: Grade matching results, confidence levels, pass/fail status

### Key Design Decisions
- **Calendar-Based Selection**: Intuitive date-range selection interface
- **Template-Driven Export**: Flexible formatting based on user needs
- **Feature-Aware Packaging**: Export content adapts to enabled system features
- **Async Processing**: Non-blocking export generation for large datasets
- **Temporary File Management**: Secure download with automatic cleanup
- **Audit Trail**: Complete logging of export activities for compliance

### Success Criteria
- User can easily select test data using calendar interface
- Export templates provide appropriate formatting for different use cases
- Generated files include all relevant data based on feature settings
- Export process completes successfully for selected data range
- Download files are properly formatted and include integrity verification
- All export activities logged for regulatory compliance

---

## Use Case 11: Calibration/Drift Check

### Overview
Periodic calibration procedure to correct for hardware drift using known reference samples. System guides user through calibration process, measures reference sample, calculates slope/offset corrections, and applies corrections to maintain measurement accuracy over time.

### Sequence Flow

```
Actor: User
UI Layer: User Interface
MW API: Middleware API  
HW API: Hardware API
DB: Database
```

**Step-by-Step API Flow:**

### Calibration Initiation

1. **Calibration Menu Access**
   ```
   User -> UI: Navigate to settings/maintenance section
   UI -> MW API: GET /api/calibration/status (async)
   MW API: Check calibration schedule and last calibration date
   MW API -> UI: Return calibration_status (current/overdue/due, last_date, next_due)
   UI: Display calibration status indicator and menu options
   ```

2. **Calibration Start Request**
   ```
   User -> UI: Selects "Start Calibration/Drift Check"
   UI -> MW API: POST /api/calibration/start (async)
   MW API: Validate system ready for calibration (idle state)
   MW API: Load reference sample specifications and expected values
   MW API -> UI: startCalibrationWizard(reference_sample_info, procedure_steps)
   UI: Display step-by-step calibration instructions
   ```

3. **Reference Sample Preparation**
   ```
   UI: Guide user through sample placement procedure
   UI: Display reference sample information and positioning instructions
   User -> UI: Confirms reference sample is properly positioned
   UI -> MW API: POST /api/calibration/sample-ready (async)
   ```

### Calibration Measurement

4. **Calibration Measurement Sequence**
   ```
   MW API -> HW API: startCalibrationMeasurement(reference_sample_id) (sync)
   HW API: Apply current instrument settings (current mode configuration)
   HW API: Execute calibration measurement sequence on reference sample
   HW API: Collect raw spectral data from known reference sample
   HW API: Monitor hardware stability during measurement
   ```

5. **Real-time Progress Updates**
   ```
   HW API -> MW API: measurementProgress(progress_percent, current_data) (async)
   MW API -> UI: updateCalibrationProgress(progress, live_spectrum) (async)
   UI: Display real-time measurement progress and live spectrum
   ```

6. **Measurement Completion**
   ```
   HW API -> MW API: Return measurement_data (spectrum, metadata, stability_check)
   ```

### Calibration Analysis

7. **Reference Sample Analysis**
   ```
   MW API: Apply current calibration algorithms to measured spectrum data
   MW API: Extract peak areas and apply current slope/offset corrections
   MW API: Generate measured_concentrations (elements, values, uncertainties)
   ```

8. **Drift Calculation and Correction**
   ```
   MW API: Compare measured vs. expected concentrations for reference sample
   MW API: Calculate slope and offset corrections for each element
   MW API: Calculate drift values and validate against acceptable thresholds
   MW API: Determine calibration pass/fail status
   ```

9. **Calibration Validation**
   ```
   MW API: Validate drift corrections are within acceptable limits
   MW API: Check measurement repeatability and stability
   MW API: Generate calibration correction factors
   ```

### Calibration Application

10. **Apply Corrections (if calibration passes)**
    ```
    MW API -> HW API: applyCalibrationCorrections(slope_offset_factors) (sync)
    HW API: Update internal calibration parameters
    HW API: Validate correction factors applied successfully
    HW API -> MW API: Return calibration_update_status (SUCCESS/FAILED)
    ```

11. **Calibration Results Display**
    ```
    MW API -> UI: displayCalibrationResults(pass_fail, drift_values, corrections) (async)
    UI: Show calibration results (pass/fail status)
    UI: Display drift values and applied correction factors
    UI: Show next calibration due date
    UI: Provide calibration certificate/report option
    ```

12. **Calibration Logging and Scheduling**
    ```
    MW API -> DB: logCalibrationEvent(calibration_results, corrections, timestamp)
    MW API: Update calibration schedule (set next due date)
    MW API: Reset calibration status to "current"
    ```

### Calibration Failure Handling

13. **Failure Scenario (if drift exceeds thresholds)**
    ```
    MW API -> UI: displayCalibrationFailure(failure_reason, recommended_action)
    UI: Alert user of calibration failure
    UI: Display recommended actions (retry, service required, check sample)
    UI: Offer retry option or escalation to service mode
    ```

### API Sequence Diagram

```
User    UI       MW API    HW API    DB
 |       |         |         |       |
 |------>|         |         |       |  1. Navigate to calibration
 |       |-------->|         |       |  2. GET /api/calibration/status
 |       |<--------|         |       |  3. Return calibration_status
 |------>|         |         |       |  4. Start calibration
 |       |-------->|         |       |  5. POST /api/calibration/start
 |       |<--------|         |       |  6. startCalibrationWizard()
 |       | [guide user]      |       |  7. Display instructions
 |------>|         |         |       |  8. Sample ready confirmation
 |       |-------->|         |       |  9. POST /api/calibration/sample-ready
 |       |         |-------->|       | 10. startCalibrationMeasurement()
 |       |         |         | [measure] | 11. Execute measurement
 |       |         |<--------|       | 12. Return measurement_data
 |       |         | [analyze spectrum] | 13. MW API calculates concentrations
 |       |         | [calc corrections] | 14. Calculate drift & corrections
 |       |         |-------->|       | 15. applyCalibrationCorrections()
 |       |         |<--------|       | 16. Return update_status
 |       |<--------|         |       | 17. displayCalibrationResults()
 |       | [show results]    |       | 18. Display pass/fail & corrections
 |       |         |---------------->| 19. logCalibrationEvent()
 |       |         |         |       |
```

### Reference Sample Management
- **Known Samples**: Certified reference materials with known concentrations
- **Sample Types**: Mode-specific reference samples (PMI vs reCycling standards)
- **Expected Values**: Database of certified element concentrations
- **Traceability**: Reference sample certificates and batch tracking

### Drift Correction Methodology
- **Slope Correction**: Adjusts sensitivity/gain for each element
- **Offset Correction**: Adjusts baseline/zero point for each element
- **Element-Specific**: Individual corrections for each measured element
- **Validation**: Ensures corrections are within acceptable scientific limits

### Calibration Scheduling
- **Time-Based**: Scheduled calibrations (daily, weekly, monthly)
- **Usage-Based**: Calibration after X number of measurements
- **Drift-Based**: Automatic calibration when drift detected
- **Regulatory**: Compliance-driven calibration requirements

### Key Design Decisions
- **User-Guided Process**: Step-by-step wizard guides users through procedure
- **Real-Time Feedback**: Live progress and spectrum display during measurement
- **Automatic Correction**: System automatically applies validated corrections
- **Failure Handling**: Clear failure reasons and recommended actions
- **Audit Trail**: Complete logging for regulatory compliance
- **Schedule Management**: Automatic tracking of calibration due dates

### Success Criteria
- User successfully completes calibration procedure with guidance
- Reference sample measured accurately with stable results
- Drift calculations completed and validated against thresholds
- Correction factors applied successfully to future measurements
- Calibration results properly logged for audit compliance
- Next calibration schedule updated appropriately

---

## Use Case 12: ConfigureXRFsettings (Feature, Settings)

### Overview
Comprehensive XRF system configuration interface that provides generic settings management across multiple feature categories including mode configuration, beam parameters, and data storage options. System manages hierarchical settings with validation, dependency resolution, and hardware constraint enforcement while maintaining settings persistence and synchronization across all system components.

### Sequence Flow

```
Actor: User
UI Layer: User Interface
MW API: Middleware API  
HW API: Hardware API
DB: Database
```

**Step-by-Step API Flow:**

### XRF Settings Access

1. **Settings Navigation**
   ```
   User -> UI: Navigate to Settings menu
   UI: Display settings interface with XRF Configuration section
   UI: Show "XRF Settings" button with current configuration status
   ```

2. **Configuration Interface Request**
   ```
   User -> UI: Press "XRF Settings" button
   UI -> MW API: GET /api/xrf/settings-overview (async)
   MW API: Retrieve current settings for all feature categories
   MW API -> DB: getCurrentXRFSettings(feature_categories)
   DB -> MW API: Return current_settings (mode, beam, datastorage, dependencies)
   MW API -> UI: Return settings_overview (categories, current_values, constraints)
   UI: Display XRF settings dashboard with feature categories
   ```

### Feature Category Selection

3. **Settings Category Display**
   ```
   UI: Show settings organized by feature categories:
   UI: - Mode Configuration (PMI, reCycling, Mining, Jewelry)
   UI: - Beam Configuration (Voltage, Current, Filters, Single/Dual Beam)
   UI: - Data Storage (Export Destinations, File Formats, Retention)
   UI: Display current active settings status for each category
   ```

4. **Feature Category Selection**
   ```
   User -> UI: Selects specific feature category (Mode/Beam/DataStorage)
   UI -> MW API: GET /api/xrf/settings/{category} (category_name) (async)
   ```

### Mode Configuration Process (If Mode Selected)

5. **Mode Settings Retrieval**
   ```
   MW API -> DB: getModeSettings(current_mode, available_modes)
   DB -> MW API: Return mode_settings (PMI, reCycling, future_modes, dependencies)
   MW API: Include mode-specific constraints and hardware requirements
   MW API -> UI: Return mode_configuration (current_mode, options, constraints)
   UI: Display mode selection interface with configuration options
   UI: Show mode-specific settings (calibration, algorithms, UI features)
   ```

6. **Mode Settings Modification**
   ```
   User -> UI: Modifies mode configuration settings
   UI: Update setting values and validate input constraints
   UI: Show setting dependencies and affected components
   User -> UI: Confirms mode configuration changes
   UI -> MW API: PUT /api/xrf/settings/mode (modified_settings) (async)
   ```

### Beam Configuration Process (If Beam Selected)

7. **Beam Settings Retrieval**
   ```
   MW API -> DB: getBeamSettings(current_configuration)
   DB -> MW API: Return beam_settings (voltage, current, filters, beam_mode)
   MW API -> HW API: getHardwareConstraints(beam_parameters)
   HW API -> MW API: Return hardware_limits (voltage_range, current_limits, filter_positions)
   MW API -> UI: Return beam_configuration (settings, limits, single_dual_options)
   UI: Display beam configuration interface with hardware constraints
   ```

8. **Beam Settings Modification**
   ```
   User -> UI: Adjusts beam parameters (voltage, current, filter positions)
   UI: Validate settings against hardware constraints in real-time
   UI: Show beam mode selection (single beam vs dual beam configuration)
   User -> UI: Confirms beam configuration changes
   UI -> MW API: PUT /api/xrf/settings/beam (modified_beam_settings) (async)
   ```

### Data Storage Configuration Process (If DataStorage Selected)

9. **Data Storage Settings Retrieval**
   ```
   MW API -> DB: getDataStorageSettings(current_policies)
   DB -> MW API: Return storage_settings (export_destinations, formats, retention)
   MW API -> UI: Return datastorage_configuration (destinations, formats, policies)
   UI: Display data storage configuration interface
   UI: Show export destinations, file formats, and retention policies
   ```

10. **Data Storage Settings Modification**
    ```
    User -> UI: Configures data storage options
    UI: Show export destination options (USB, SD, Cloud, Network)
    UI: Display file format selection (CSV, JSON, PDF, XML)
    UI: Present retention policy configuration (duration, automatic cleanup)
    User -> UI: Confirms data storage configuration changes
    UI -> MW API: PUT /api/xrf/settings/datastorage (modified_storage_settings) (async)
    ```

### Settings Validation and Application

11. **Settings Validation Process**
    ```
    MW API: Validate modified settings for selected feature category
    MW API: Check setting dependencies and conflicts across features
    MW API: Verify hardware compatibility and constraint compliance
    MW API: Identify affected system components and required updates
    MW API: Generate validation report with warnings and errors
    ```

12. **Conflict Resolution** (If conflicts detected)
    ```
    MW API -> UI: displaySettingsConflicts(conflict_details, resolution_options)
    UI: Show conflicts dialog with detailed explanations
    UI: Present resolution options and automatic fix suggestions
    User -> UI: Selects resolution approach or manually resolves conflicts
    UI -> MW API: PUT /api/xrf/settings/resolve-conflicts (resolution_choices) (async)
    ```

13. **Hardware Configuration Application**
    ```
    MW API -> HW API: applyHardwareSettings(validated_settings) (sync)
    HW API: Validate hardware-specific setting limits and capabilities
    HW API: Apply settings to hardware components (tube, detector, filters)
    HW API: Verify settings application and report current configuration
    HW API -> MW API: Return hardware_config_status (success/failure, applied_settings)
    ```

14. **Settings Persistence and Synchronization**
    ```
    MW API -> DB: saveXRFSettings(feature_category, validated_settings, timestamp)
    MW API: Update system configuration cache
    MW API: Synchronize settings across all system components
    MW API: Update settings versioning for audit trail
    MW API -> UI: Return settings_update_status (success, applied_changes, warnings)
    ```

15. **Settings Confirmation Display**
    ```
    UI: Display settings update confirmation
    UI: Show applied changes summary and current active configuration
    UI: Update settings interface to reflect new configuration
    UI: Display any warnings or recommendations from settings application
    ```

### Settings Import/Export Process

16. **Settings Backup/Deployment** (Optional)
    ```
    User -> UI: Selects settings import/export option
    UI -> MW API: GET /api/xrf/settings/export (feature_categories) (async)
    MW API -> DB: exportSettingsPackage(selected_features, include_metadata)
    MW API -> UI: Return settings_export_package (configuration_file, metadata)
    UI: Provide download interface for settings backup file
    
    For Import:
    UI -> MW API: POST /api/xrf/settings/import (settings_package) (async)
    MW API: Validate imported settings and check compatibility
    MW API: Apply validated settings following normal validation process
    ```

### API Sequence Diagram

```
User    UI       MW API            HW API    DB
 |       |         |                 |       |
 |------>|         |                 |       |  1. Navigate to XRF Settings
 |       |-------->|                 |       |  2. GET /api/xrf/settings-overview
 |       |         |------------------------>|  3. getCurrentXRFSettings()
 |       |         |<------------------------|  4. Return current_settings
 |       |<--------|                 |       |  5. Return settings_overview
 |       | [show categories]         |       |  6. Display feature categories
 |------>|         |                 |       |  7. Select feature category
 |       |-------->|                 |       |  8. GET /api/xrf/settings/{category}
 |       |         |------------------------>|  9. getSettings(category)
 |       |         |<------------------------|  10. Return category_settings
 |       |         |---------------->|       |  11. getHardwareConstraints() [if beam]
 |       |         |<----------------|       |  12. Return hardware_limits
 |       |<--------|                 |       |  13. Return category_configuration
 |       | [show settings interface] |       |  14. Display configuration options
 |------>|         |                 |       |  15. Modify settings
 |       |-------->|                 |       |  16. PUT /api/xrf/settings/{category}
 |       |         | [validate settings]     |  17. Validate and check conflicts
 |       |         |---------------->|       |  18. applyHardwareSettings() (sync)
 |       |         |<----------------|       |  19. Return hardware_config_status
 |       |         |------------------------>|  20. saveXRFSettings()
 |       |<--------|                 |       |  21. Return settings_update_status
 |       | [show confirmation]       |       |  22. Display applied changes
 |       |         |                 |       |
```

### XRF Feature Categories

**Mode Configuration:**
- **Current Active Mode**: PMI, reCycling, or future modes (Mining, Jewelry)
- **Mode-Specific Settings**: Calibrations, algorithms, analysis parameters
- **UI Feature Toggles**: Mode-specific interface elements and workflows
- **Analysis Parameters**: Element libraries, detection limits, uncertainty calculations
- **Measurement Protocols**: Standard measurement times, beam parameters per mode

**Beam Configuration:**
- **X-ray Tube Settings**: Voltage (kV), current (μA), filament parameters
- **Filter Configuration**: Filter wheel positions, filter selection logic
- **Beam Mode**: Single beam vs dual beam operation configuration
- **Safety Parameters**: Maximum voltage/current limits, interlock settings
- **Timing Parameters**: Warm-up times, stabilization delays, measurement cycles

**Data Storage Configuration:**
- **Export Destinations**: USB, SD card, cloud services, network locations
- **File Format Preferences**: CSV, JSON, PDF, XML, raw data formats
- **Retention Policies**: Automatic cleanup schedules, archive durations
- **Backup Settings**: Automatic backup schedules, redundancy options
- **Compression Options**: Data compression settings for storage efficiency

### Settings Hierarchy and Dependencies

**Hierarchical Structure:**
- **System Level**: Global settings affecting entire system
- **Mode Level**: Settings specific to PMI, reCycling, or other modes
- **Component Level**: Hardware-specific settings and constraints
- **User Level**: User preferences and customizations

**Dependency Management:**
- **Cross-Feature Dependencies**: Mode settings affecting beam configuration
- **Hardware Constraints**: Physical limitations enforcing setting boundaries
- **Safety Dependencies**: Settings interactions affecting radiation safety
- **Calibration Dependencies**: Settings requiring recalibration when changed

### Settings Validation and Constraints

**Validation Rules:**
- **Range Validation**: Settings within acceptable minimum/maximum values
- **Hardware Compatibility**: Settings compatible with installed hardware
- **Safety Compliance**: Settings meeting radiation safety requirements
- **Mode Compatibility**: Settings appropriate for selected operation mode

**Constraint Enforcement:**
- **Hardware Limits**: Physical constraints from X-ray tube and detector
- **Safety Limits**: Maximum safe operating parameters
- **Calibration Requirements**: Settings requiring calibration validation
- **Performance Constraints**: Settings maintaining system performance standards

### Settings Persistence and Versioning

**Persistence Management:**
- **Database Storage**: Settings stored with versioning and timestamps
- **Configuration Cache**: In-memory cache for fast settings access
- **Backup Integration**: Settings included in system backup procedures
- **Migration Support**: Settings migration during software updates

**Version Control:**
- **Change Tracking**: Complete audit trail of settings modifications
- **Rollback Capability**: Ability to revert to previous settings configurations
- **Configuration Templates**: Predefined settings packages for deployment
- **User Profiles**: Settings profiles for different users or applications

### Settings Import/Export Features

**Export Capabilities:**
- **Full Configuration**: Complete system settings backup
- **Selective Export**: Export specific feature categories
- **Template Creation**: Create deployment templates for multiple units
- **Metadata Inclusion**: Settings context and validation information

**Import Capabilities:**
- **Configuration Deployment**: Deploy settings to multiple systems
- **Template Application**: Apply predefined configuration templates
- **Validation and Compatibility**: Verify imported settings compatibility
- **Merge Options**: Merge imported settings with current configuration

### Key Design Decisions
- **Generic Interface**: Unified settings interface supporting multiple feature categories
- **Hierarchical Management**: Settings organized by system, mode, component, and user levels
- **Comprehensive Validation**: Multi-level validation including hardware constraints
- **Real-time Feedback**: Immediate validation and conflict resolution during configuration
- **Hardware Integration**: Direct hardware constraint checking and application
- **Audit Trail**: Complete settings change logging for compliance and troubleshooting

### Success Criteria
- Users can easily navigate and configure all XRF feature categories
- Settings validation prevents invalid configurations and hardware damage
- Conflict resolution provides clear guidance for resolving setting dependencies
- Hardware constraints are enforced in real-time during settings modification
- Settings persistence maintains configuration across power cycles and updates
- Import/export functionality enables settings backup and deployment
- Settings changes are properly synchronized across all system components
- Complete audit trail maintains configuration change history for compliance

---
