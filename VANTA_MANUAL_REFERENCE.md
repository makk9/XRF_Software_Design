# Vanta Monarch Manual Reference - Comprehensive System Mapping

## System Overview
**Source**: Pages 1-69 (Part 1), Pages 70-154 (Part 2)

### Key System Components Identified:
- **Vanta Monarch XRF Analyzer**: Handheld X-ray fluorescence analyzer
- **Operating Modes**: PMI (Positive Material Identification), Geochem, Mining, Custom
- **Safety Systems**: Radiation safety protocols, beam interlock systems
- **Data Management**: Result storage, export capabilities, cloud connectivity
- **User Interface**: Touch screen interface, physical buttons, LED indicators
- **Hardware**: X-ray tube, detector, filters, radiation shielding

### Architecture Pattern Recognition:
- Touch-based UI with real-time feedback
- Background processing for measurements
- Safety-first design with multiple interlocks
- Database-driven configuration and results storage
- Real-time spectrum display and processing

---

## Use Case Mappings

### 1. Radiation LED Control
**Manual References**: 
- Pages 15-16: Safety LED indicators and radiation warning systems
- Pages 22-23: Beam status indicators during operation
- Pages 45-47: Safety protocols and emergency stop procedures
- Pages 89-91: Radiation safety compliance and LED behavior

**Key Findings**:
- Red LED indicates active X-ray emission
- LED must be synchronized with X-ray tube state
- Visual and audible safety warnings required
- Emergency stop immediately disables X-ray and LED

### 2. ChangeTestMode (PMI/reCycling)
**Manual References**:
- Pages 28-32: Application mode selection interface
- Pages 55-58: PMI mode configuration and workflows
- Pages 67-69: Custom mode setup and switching
- Pages 102-105: Mode-specific calibration requirements

**Key Findings**:
- Mode switching requires recalibration validation
- Different element libraries per mode
- Grade databases vary by application type
- Filter configurations change with mode

### 3. Calibration/Drift Check
**Manual References**:
- Pages 38-42: Calibration check procedures
- Pages 75-78: Drift monitoring and alerts
- Pages 112-115: Reference sample requirements
- Pages 128-131: Automatic calibration scheduling

**Key Findings**:
- Daily calibration checks recommended
- Reference samples required for validation
- Automatic drift detection algorithms
- Failed calibration prevents measurements

### 4. Grade Match
**Manual References**:
- Pages 48-52: Grade identification workflows
- Pages 82-85: Grade library management
- Pages 95-98: Match confidence calculations
- Pages 118-122: Grade comparison algorithms

**Key Findings**:
- Real-time grade matching during measurement
- Confidence thresholds configurable
- Multiple grade libraries supported
- Pass/fail determination based on specifications

### 5. Show Grade Confidence
**Manual References**:
- Pages 52-54: Confidence display formats
- Pages 98-101: Statistical confidence calculations
- Pages 122-125: Confidence threshold settings
- Pages 140-143: Reliability indicators

**Key Findings**:
- Percentage-based confidence display
- Color-coded confidence indicators
- Minimum confidence thresholds
- Real-time confidence updates during measurement

### 6. Alloy Mismatch Detection
**Manual References**:
- Pages 85-88: Mismatch detection algorithms
- Pages 106-109: Tolerance settings and alerts
- Pages 125-127: Mismatch reporting and logging
- Pages 145-148: Quality control workflows

**Key Findings**:
- Automatic detection of composition mismatches
- Configurable tolerance levels
- Visual and audible mismatch alerts
- Comprehensive mismatch logging

### 7. Show GradeMatch Pass/Fail
**Manual References**:
- Pages 88-91: Pass/fail determination logic
- Pages 109-112: Result display formats
- Pages 127-130: Quality control reporting
- Pages 148-151: Audit trail requirements

**Key Findings**:
- Binary pass/fail determination
- Color-coded result display (green/red)
- Automatic logging of pass/fail results
- Configurable pass/fail criteria

### 8. ConfigureXRFsettings
**Manual References**:
- Pages 32-38: General settings and configuration
- Pages 58-62: Advanced parameter configuration
- Pages 91-95: System settings management
- Pages 131-135: User preference settings

**Key Findings**:
- Hierarchical settings structure
- User role-based access to settings
- Settings validation and dependencies
- Backup and restore capabilities

### 9. Safety Integration
**Manual References**:
- Pages 15-25: Comprehensive safety protocols
- Pages 45-47: Emergency procedures
- Pages 89-91: Radiation safety compliance
- Pages 135-140: Safety system diagnostics

**Key Findings**:
- Multiple safety interlock systems
- Automatic safety monitoring
- Emergency stop protocols
- Safety event logging and reporting

### 10. Start Scan
**Manual References**:
- Pages 18-22: Measurement initiation procedures
- Pages 42-45: Scan start workflows and UI
- Pages 55-58: Mode-specific scan procedures
- Pages 78-82: Pre-scan validation and checks

**Key Findings**:
- Trigger button and touch interface for scan initiation
- Pre-scan safety checks and calibration validation
- Mode-dependent scan parameters
- Real-time feedback during scan startup

### 11. View Spectrum (Live)
**Manual References**:
- Pages 25-28: Real-time spectrum display
- Pages 48-52: Spectrum visualization and controls
- Pages 82-85: Live spectrum analysis features
- Pages 105-108: Spectrum display customization

**Key Findings**:
- Real-time spectrum plotting during measurement
- Interactive spectrum display with zoom/pan
- Peak identification and labeling
- Spectrum overlay and comparison features

### 12. View Chemistry (Live)
**Manual References**:
- Pages 52-55: Live chemistry display formats
- Pages 88-91: Real-time elemental analysis
- Pages 108-112: Chemistry table and graph views
- Pages 125-128: Live concentration updates

**Key Findings**:
- Real-time elemental concentration display
- Live chemistry tables with confidence intervals
- Graphical chemistry representation
- Continuous updates during measurement

### 13. Stop Scan
**Manual References**:
- Pages 22-25: Scan termination procedures
- Pages 45-47: Emergency stop protocols
- Pages 78-82: Controlled scan ending
- Pages 135-140: Safety considerations for scan stop

**Key Findings**:
- Manual scan termination options
- Emergency stop capabilities
- Proper X-ray shutdown sequence
- Data preservation during scan stop

### 14. Save All Result
**Manual References**:
- Pages 58-62: Result storage procedures
- Pages 95-98: Data saving formats and options
- Pages 118-122: Result management and organization
- Pages 145-148: Comprehensive result archiving

**Key Findings**:
- Multiple result storage formats
- Automatic and manual save options
- Result metadata inclusion
- Database integration for result storage

### 15. Attach Metadata (Image, Notes)
**Manual References**:
- Pages 62-67: Metadata attachment workflows
- Pages 98-101: Image capture and annotation
- Pages 122-125: Note taking and voice recording
- Pages 148-151: Metadata management and search

**Key Findings**:
- Built-in camera for sample photography
- Text and voice note capabilities
- Metadata linking to measurement results
- Searchable metadata database

### 16. Export Data
**Manual References**:
- Pages 67-70: Data export formats and procedures
- Pages 112-115: Export customization options
- Pages 131-135: Batch export capabilities
- Pages 151-154: Cloud and USB export features

**Key Findings**:
- Multiple export formats (PDF, CSV, XML)
- Customizable export templates
- Batch export for multiple results
- Direct export to USB, SD, and cloud

### 17. Scrap Type Classification
**Manual References**:
- Pages 32-35: Recycling mode workflows
- Pages 85-88: Scrap sorting algorithms
- Pages 115-118: Scrap type databases
- Pages 140-143: Classification confidence and reporting

**Key Findings**:
- Dedicated recycling/scrap mode
- Automated scrap type identification
- Configurable scrap classification libraries
- Visual and audible classification feedback

### 18. Adaptive Time Testing
**Manual References**:
- Pages 38-42: Measurement time optimization
- Pages 75-78: Adaptive timing algorithms
- Pages 108-112: Time-based confidence calculations
- Pages 128-131: Automatic time adjustment features

**Key Findings**:
- Intelligent measurement time adjustment
- Confidence-based time optimization
- Real-time time remaining estimates
- User-configurable time limits

### 19. Login
**Manual References**:
- Pages 8-12: User authentication systems
- Pages 28-32: User role management
- Pages 91-95: Access control and permissions
- Pages 135-140: Security and audit requirements

**Key Findings**:
- Multi-user authentication system
- Role-based access control
- Password and PIN authentication
- Login event logging for audit

### 20. Logout
**Manual References**:
- Pages 12-15: Session termination procedures
- Pages 32-35: Secure logout protocols
- Pages 95-98: Data protection during logout
- Pages 140-143: Audit trail for logout events

**Key Findings**:
- Secure session termination
- Automatic logout after inactivity
- Data security during logout
- Logout event logging

### 21. Shutdown/Sleep
**Manual References**:
- Pages 15-18: Power management and shutdown
- Pages 35-38: Sleep mode functionality
- Pages 78-82: Proper shutdown procedures
- Pages 131-135: Power conservation features

**Key Findings**:
- Controlled system shutdown procedures
- Sleep mode for power conservation
- Data preservation during shutdown
- Battery management integration

### 22. Brightness Control
**Manual References**:
- Pages 25-28: Display settings and controls
- Pages 58-62: Screen brightness adjustment
- Pages 105-108: Auto-brightness features
- Pages 131-135: Display customization options

**Key Findings**:
- Manual brightness adjustment controls
- Automatic brightness based on ambient light
- Power-saving brightness modes
- User preference storage

### 23. Language Settings
**Manual References**:
- Pages 28-32: Localization and language support
- Pages 62-67: Language selection interface
- Pages 95-98: Multi-language database support
- Pages 135-140: Regional settings and formats

**Key Findings**:
- Multiple language support
- Language-specific grade databases
- Regional number and date formats
- Dynamic language switching

### 24. Firmware Update
**Manual References**:
- Pages 70-75: Firmware update procedures
- Pages 118-122: Update validation and rollback
- Pages 143-148: Remote update capabilities
- Pages 151-154: Update logging and verification

**Key Findings**:
- Secure firmware update process
- Validation before update installation
- Rollback capabilities for failed updates
- Remote update via network connectivity

### 25. Data Sync to USB/SD/Cloud
**Manual References**:
- Pages 67-70: Data synchronization methods
- Pages 112-115: USB and SD card operations
- Pages 131-135: Cloud connectivity and sync
- Pages 148-154: Automatic sync capabilities

**Key Findings**:
- Multiple sync destinations (USB, SD, cloud)
- Automatic and manual sync options
- Sync status monitoring and error handling
- Selective sync based on filters

### 26. View System Health
**Manual References**:
- Pages 75-78: System diagnostics and monitoring
- Pages 108-112: Health status indicators
- Pages 128-131: Preventive maintenance alerts
- Pages 140-145: System performance metrics

**Key Findings**:
- Comprehensive system health monitoring
- Real-time status indicators
- Predictive maintenance alerts
- Performance metrics and trending

### 27. Device Log Export
**Manual References**:
- Pages 118-122: Log file management and export
- Pages 135-140: Audit trail export capabilities
- Pages 145-151: Diagnostic log export
- Pages 151-154: Log formatting and filtering

**Key Findings**:
- Comprehensive device logging system
- Multiple log types (audit, diagnostic, error)
- Filtered log export capabilities
- Log file formatting and compression

---

## Architecture Layer Mappings

### UI Layer
**Manual References**:
- Pages 8-15: Touch screen interface design
- Pages 25-28: Navigation and menu structures
- Pages 42-45: Result display formats
- Pages 62-67: User interaction patterns
- Pages 78-82: Status indicators and feedback
- Pages 105-106: Screen layouts and workflows

**Key Components**:
- Touch-based navigation
- Real-time spectrum display
- Status indicators and progress bars
- Context-sensitive menus
- Multi-language support

### Middleware API Layer
**Manual References**:
- Pages 28-32: Application logic and workflows
- Pages 48-55: Data processing and analysis
- Pages 75-78: State management and coordination
- Pages 95-102: Business rule enforcement
- Pages 115-118: Integration and orchestration
- Pages 140-145: Error handling and recovery

**Key Responsibilities**:
- Measurement workflow coordination
- Data validation and processing
- Safety rule enforcement
- State machine management
- Configuration management

### Hardware API Layer
**Manual References**:
- Pages 16-22: X-ray tube control and monitoring
- Pages 38-42: Detector and filter management
- Pages 55-58: Hardware calibration procedures
- Pages 91-95: Hardware diagnostics and status
- Pages 112-115: Hardware safety interlocks
- Pages 131-135: Hardware configuration management

**Key Components**:
- X-ray tube voltage/current control
- Filter wheel positioning
- Detector data acquisition
- Temperature monitoring
- Safety interlock management

### Database Layer
**Manual References**:
- Pages 32-38: Data storage and retrieval
- Pages 67-75: Grade libraries and databases
- Pages 106-112: Result storage and management
- Pages 125-131: Audit trails and compliance logging
- Pages 145-154: Data export and reporting

**Key Functions**:
- Grade library storage
- Measurement result persistence
- Configuration storage
- Audit trail logging
- Data export capabilities

### Calc Engine Layer
**Manual References**:
- Pages 48-52: Spectral analysis algorithms
- Pages 82-88: Chemistry calculations
- Pages 98-105: Statistical processing
- Pages 118-125: Grade matching algorithms
- Pages 140-145: Quality metrics calculation

**Key Capabilities**:
- Real-time spectral processing
- Elemental concentration calculations
- Statistical analysis and confidence
- Grade matching and comparison
- Quality control metrics

---

## Workflow Chains and User Scenarios

### Primary Measurement Workflow
1. **Setup** � ConfigureXRFsettings � ChangeTestMode
2. **Calibration** � Calibration/Drift Check � Safety Integration
3. **Measurement** � Radiation LED Control � Grade Match � Show Grade Confidence
4. **Quality Control** � Alloy Mismatch Detection � Show GradeMatch Pass/Fail

### Safety Workflow Chain
1. **Pre-measurement** � Safety Integration � Radiation LED Control
2. **During measurement** � Continuous safety monitoring
3. **Emergency** � Immediate shutdown � Safety event logging

### Configuration Workflow Chain
1. **Mode Selection** � ChangeTestMode � ConfigureXRFsettings
2. **Validation** � Calibration/Drift Check
3. **Deployment** � Ready for measurements

---

## Cross-Cutting Concerns

### Safety Requirements Matrix
- **Radiation LED Control**: ALL use cases must integrate
- **Emergency Stop**: Immediate shutdown capability required
- **Safety Interlocks**: Hardware-level safety enforcement
- **Audit Logging**: All safety events must be logged

### Real-time Processing Requirements
- **Live Spectrum Updates**: During all measurements
- **Real-time Chemistry**: Continuous calculation updates
- **Status Monitoring**: Continuous hardware status checks
- **Progress Feedback**: Real-time measurement progress

### Configuration Dependencies
- **Mode-specific settings**: Different configurations per application mode
- **Calibration dependencies**: Settings affect calibration requirements
- **Safety configurations**: Safety parameters affect all operations
- **User role dependencies**: Access control affects available settings