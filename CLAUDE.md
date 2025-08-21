# XRF Analyzer System Design Project

## Project Overview
- Designing comprehensive software architecture for XRF (X-Ray Fluorescence) analyzer system
- Safety-critical system for material analysis using X-ray radiation
- Following structured UI → Middleware API → Hardware API → Database → Safety Integration pattern
- Focus on regulatory compliance, audit trails, and radiation safety

## System Architecture Pattern
All use cases follow this standardized structure:
- **UI Layer**: User interface, displays, controls, visual feedback
- **Middleware API**: Business logic, data processing, state management, coordination
- **Hardware API**: Direct hardware control and monitoring (X-ray tube, DPP, filters, LED)
- **Database**: Data persistence, audit trails, configuration storage, compliance logging
- **Safety/Quality Integration**: Safety interlocks, validation, emergency stops
- **Calc Engine**: Chemistry calculations and spectral analysis

## Key Technical Patterns
- **State Management**: MW API maintains system state machine and coordinates all transitions
- **Safety Integration**: All operations must integrate with safety interlocks and emergency stops
- **Database Logging**: Comprehensive audit trails required for regulatory compliance
- **Hardware Abstraction**: HW API provides consistent interface to physical components
- **Configuration Management**: Centralized settings with validation and dependency handling
- **Real-time Processing**: Live updates during measurements with statistical processing

## Safety Requirements (CRITICAL)
- **Radiation LED Control**: Mandatory safety indicator - LED ON during X-ray emission, OFF when stopped
- **Emergency Stop Integration**: All operations must support immediate emergency shutdown
- **Cross-validation**: LED failure triggers emergency stop, validate tube state consistency
- **Audit Compliance**: All safety events must be logged with timestamps and user context

## Completed Use Cases
1. **Radiation LED Control** - Safety indicator for X-ray emission status
2. **ChangeTestMode** - PMI/reCycling mode switching with HW/SW configuration
3. **Calibration/Drift Check** - Periodic calibration using reference samples
4. **Grade Match** - Compare measured chemistry against grade library specifications
5. **Show Grade Confidence** - Statistical reliability display for grade identifications
6. **Alloy Mismatch Detection** - Detect samples not matching grade specifications
7. **Show GradeMatch Pass/Fail** - Automated Pass/Fail determination for QC
8. **ConfigureXRFsettings** - Generic configuration system for XRF features

## Design Conventions
- Use sequence diagrams for API flow documentation
- Document all component interactions with specific API method names
- Include error handling and rollback procedures in all flows
- Specify async vs sync API calls clearly
- Show data structures passed between components
- Include timing considerations and real-time constraints
- Document state transitions and validation points

## Application Modes
- **PMI (Positive Material Identification)**: Material verification and grade matching
- **reCycling**: Scrap sorting and material classification
- **Future**: Mining, jewelry applications (extensible architecture)

## Hardware Components
- **X-ray Tube**: Voltage/current control, emission timing
- **DPP (Digital Pulse Processor)**: Spectrum data collection
- **Radiation LED**: Safety indicator (hardware-controlled)
- **Filters**: Automated filter wheel positioning
- **Detector**: Spectral measurement and data acquisition

## Data Flow Patterns
- **Live Spectrum**: HW API → MW API → UI (real-time spectrum updates)
- **Live Chemistry**: HW API → MW API → Calc Engine → MW API → UI
- **Configuration**: UI → MW API → Validation → HW API → Database
- **Safety Events**: HW API → MW API → Safety System → Database + Emergency Actions

## Current Phase
- Creating detailed sequence API flow diagrams for each use case
- Documenting component interactions, data structures, and error handling
- Preparing technical blueprint for implementation phase

## Next Steps
- Complete sequence diagrams for all use cases
- Define specific API contracts and data structures
- Document error scenarios and recovery procedures
- Validate safety integration points