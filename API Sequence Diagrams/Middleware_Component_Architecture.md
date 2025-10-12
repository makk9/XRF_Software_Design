# Middleware Component Architecture

## Overview
This document defines the logical middleware components for the XRF analyzer system. Each component groups cohesive API functions based on single responsibility principle, practical boundaries, and implementation considerations. These components form the middleware layer between the UI and Hardware/Database layers.

**Architecture Pattern**: UI Layer → Middleware Components → Hardware API / Database / CalcEngine

---

## Component Index

1. [Hardware Interface Manager](#1-hardware-interface-manager)
2. [Safety & Interlock Manager](#2-safety--interlock-manager)
3. [Test Execution Manager](#3-test-execution-manager)
4. [Method Manager](#4-method-manager)
5. [Calc Engine Manager](#5-calc-engine-manager)
6. [Grade Library Manager](#6-grade-library-manager)
7. [Results Manager](#7-results-manager)
8. [Export Manager](#8-export-manager)
9. [Network Manager](#9-network-manager)
10. [Settings Manager](#10-settings-manager)
11. [Device Manager](#11-device-manager)
12. [Session Manager](#12-session-manager)
13. [Power Manager](#13-power-manager)
14. [Diagnostics Manager](#14-diagnostics-manager)
15. [Update Manager](#15-update-manager)
16. [Compliance Manager](#16-compliance-manager)
17. [Notification Manager](#17-notification-manager)

---

## 1. Hardware Interface Manager

**Responsibility**: Direct control and communication with hardware components (DPP, X-ray tube, camera hardware). Provides low-level hardware abstraction layer.

**Key Characteristics**:
- Direct hardware API calls
- Synchronous and asynchronous hardware operations
- Hardware state management
- Error handling for hardware failures

**API Functions**:

### Core Hardware Control
- `initializeHardware()` - Initialize DPP and X-ray tube for measurement
- `shutdownHardware()` - Power down DPP and X-ray tube components

### X-Ray Tube Control
- `startXRayTube(voltage, current)` - Begin X-ray emission with specified parameters
- `stopXRayTube()` - Cease X-ray emission
- `shutdownXRayTube()` - Safely power down X-ray tube

### DPP (Digital Pulse Processor) Control
- `startSpectrumAcquisition()` - Begin DPP data collection
- `stopSpectrumAcquisition()` - Stop DPP data collection
- `shutdownDPP()` - Safely power down DPP hardware
- `retrieveFinalSpectrum()` - Get completed spectrum data from DPP
- `streamLiveSpectrum()` - Continuously stream real-time spectrum data from DPP

### Camera Hardware Control
- `initializeCamera(cameraType)` - Initialize specific camera hardware (aiming, panoramic)
- `shutdownCamera()` - Power down camera hardware
- `startCameraPreview(cameraType)` - Begin live camera preview stream
- `stopCameraPreview()` - Stop camera preview and release camera hardware
- `switchCamera(cameraType)` - Switch between aiming and panoramic cameras
- `captureImage(cameraType)` - Capture still image from active camera
- `controlIRCollimator(enabled)` - Turn IR laser targeting on/off

### Power Control
- `powerUpEssentialComponents()` - Restore power to CPU, display, essential hardware (wake)

**Dependencies**:
- Hardware API (DPP, Tube, Camera drivers)
- Safety & Interlock Manager (for safety validation before hardware operations)

---

## 2. Safety & Interlock Manager

**Responsibility**: Enforce all safety rules, interlocks, and emergency shutdown procedures. Monitors radiation LED, trigger modes, and safety conditions. This is a **CRITICAL SAFETY COMPONENT**.

**Key Characteristics**:
- Safety-critical operations
- Real-time monitoring
- Emergency stop coordination
- Audit logging for compliance

**API Functions**:

### Radiation LED Control (Critical Safety)
- `activateRadiationLED()` - Turn on radiation warning LED synchronized with X-ray tube
- `deactivateRadiationLED()` - Turn off radiation warning LED synchronized with tube stop
- `monitorLEDIntegrity()` - Continuously check LED circuit status during operation
- `validateLEDTubeSync()` - Cross-check LED state matches X-ray tube state
- `logLEDEvent(eventType, timestamp)` - Record LED events for compliance

### Safety Trigger Modes
- `getSafetyTriggerMode()` - Retrieve configured trigger mode (deadman/timed-lock/two-handed)
- `validateTriggerConditions()` - Check if trigger requirements are met before allowing test
- `monitorTriggerState()` - Continuously monitor trigger/button state during test
- `enableTriggerLock()` - Activate trigger lock after timeout interval
- `disableTriggerLock()` - Unlock trigger via System Tray action
- `abortTestOnTriggerRelease()` - Emergency stop if deadman trigger released

### Safety Interlocks
- `validateSafetyInterlocks()` - Check all safety conditions before allowing test
- `checkSamplePresent()` - Verify material is in contact with measurement window
- `checkChamberDoorClosed()` - Verify chamber door is sealed (workstation mode)
- `checkWorkstationDocked()` - Verify analyzer is properly docked (workstation mode)
- `checkHardwareOperational()` - Validate X-ray tube, DPP, and LED are functional
- `validateAllInterlocks()` - Aggregate check of all interlock conditions
- `monitorInterlocksDuringTest()` - Continuously monitor interlocks during active test
- `logInterlockViolation(interlockType, timestamp)` - Record interlock failure for audit

### Emergency Stop
- `triggerEmergencyStop()` - Immediately stop X-ray emission (LED failure, interlock violation)

**Dependencies**:
- Hardware Interface Manager (for LED control, hardware status)
- Test Execution Manager (to abort tests)
- Notification Manager (safety alerts)
- Database (audit logging)

---

## 3. Test Execution Manager

**Responsibility**: Orchestrate test execution lifecycle, manage test timing, sequencing, and batch operations. Coordinates between safety, hardware, and method managers.

**Key Characteristics**:
- Test state machine management
- Multi-test and batch orchestration
- Timing control and monitoring
- Coordination hub for test workflows

**API Functions**:

### Test Execution
- `executeTest()` - Run single test (orchestrates Start/Stop Test flow)
- `monitorTestExecution()` - Monitor timing and safety during active test
- `checkActiveTest()` - Verify if X-ray scan is currently running

### Test Timing Configuration
- `getTestTiming()` - Get configured min/max times for active method
- `getMethodBeamConfiguration(methodId)` - Retrieve number of beams and defaults
- `setBeamTiming(beamId, minTime, maxTime)` - Configure and validate min/max times
- `saveTimingConfiguration(methodId)` - Persist timing settings for method
- `extendTestTime(additionalSeconds)` - Dynamically extend test duration (adaptive mode)

### Multi-Test & Batch Sequencing
- `getMultiTestConfiguration()` - Retrieve repeat count and prompt settings
- `setRepeatTestConfiguration(count, promptEnabled)` - Configure repeat test settings
- `loadBatchTestScript(filePath)` - Import batch test sequence from PC-generated file
- `getBatchTestSequence()` - Retrieve loaded batch test steps and parameters
- `executeBatchSequence()` - Run batch test sequence automatically
- `saveBatchConfiguration()` - Persist batch/repeat settings

### Adaptive Testing
- `adjustBeamParameters(voltage, current)` - Modify X-ray tube settings during test (with safety validation)

### Calibration Testing
- `executeCalibrationTest()` - Perform test on reference sample (calls test execution flow)

**Dependencies**:
- Safety & Interlock Manager (validation before test start)
- Hardware Interface Manager (hardware control)
- Method Manager (method parameters, timing)
- Calc Engine Manager (spectrum processing)
- Results Manager (result storage)
- Notification Manager (test status updates)

---

## 4. Method Manager

**Responsibility**: Manage analysis methods, configuration parameters, user factors, compound calculations, and display preferences. Method-specific settings and calibrations.

**Key Characteristics**:
- Method selection and activation
- Method-specific configuration
- User calibration (factors, compounds)
- Display preferences per method

**API Functions**:

### Method Selection & Configuration
- `getAvailableMethods()` - Retrieve list of factory-calibrated/loaded methods
- `selectMethod(methodId)` - Set the active method for testing
- `loadMethodParameters(methodId)` - Load method configuration (elements, criteria, beams)
- `getActiveMethod()` - Retrieve currently selected method

### Method Display Preferences
- `getMethodDisplayOptions(methodId)` - Retrieve current display preferences for method
- `setDisplayOption(methodId, optionName, enabled)` - Enable/disable specific display option
- `saveDisplayPreferences(methodId)` - Persist display configuration for method

### User Factors (Site Calibration)
- `getUserFactorSets(methodId)` - Retrieve all defined user factor sets for method
- `createUserFactorSet(name, methodId)` - Create new named factor set
- `setElementFactor(factorSetId, element, multiplier, offset)` - Configure factor/offset
- `selectActiveFactorSet(factorSetId)` - Switch to different factor set
- `applyUserFactors(rawResults, factorSetId)` - Apply linear correction to raw XRF results
- `saveUserFactorSet(factorSetId)` - Persist factor set configuration

### Compound Calculations (Geochem)
- `getCompoundTemplates(methodId)` - Retrieve available compound conversion templates
- `addCompoundTemplate(methodId, element, compound)` - Add new element-to-compound conversion
- `calculateCompoundConversion(elementalResults)` - Convert elemental wt% to compound wt%
- `enableCompoundDisplay(methodId, enabled)` - Toggle compound display on/off
- `saveCompoundConfiguration(methodId)` - Persist compound calculation settings

### Adaptive Mode Configuration
- `enableAdaptiveMode(methodId, enabled)` - Enable/disable adaptive testing for method
- `setAdaptiveConstraints(maxTime, maxVoltage, maxCurrent)` - Configure adaptive limits

**Dependencies**:
- Database (method storage, configuration persistence)
- Calc Engine Manager (for user factor application, compound conversion)
- Notification Manager (method change notifications)

---

## 5. Calc Engine Manager

**Responsibility**: Interface to Calculation Engine (CalcEngine) process for spectral analysis, chemistry calculations, and spectrum quality assessment. Manages CalcEngine communication.

**Key Characteristics**:
- Inter-process communication with CalcEngine
- Spectrum data processing
- Real-time quality assessment
- Chemistry calculation coordination

**API Functions**:

### Chemistry Calculation
- `calculateChemistry(spectrum)` - Process spectrum to compute elemental concentrations via CalcEngine

### Spectrum Analysis
- `analyzeSpectrumQuality(liveSpectrum)` - Send live spectrum to CalcEngine for quality assessment
- `getLightElementProbability()` - Retrieve CalcEngine detection probability for light elements
- `identifyPeaks(liveSpectrum)` - Automatically detect and mark significant element peaks

### Data Processing
- `calculateCompoundConversion(elementalResults)` - Convert elemental wt% to compound wt% (also called from Method Manager)
- `applyUserFactors(rawResults, factorSetId)` - Apply linear correction (also called from Method Manager)
- `calculateAverageResults(resultIds)` - Compute mean composition for multi-test sequences

**Dependencies**:
- CalcEngine Process (separate process for calculations)
- Hardware Interface Manager (spectrum data source)
- Method Manager (method-specific parameters, user factors)

---

## 6. Grade Library Manager

**Responsibility**: Manage grade libraries, perform grade matching, and handle scrap classification. Library activation and grade comparison logic.

**Key Characteristics**:
- Grade library management
- Match number calculation
- Scrap classification (recycling mode)
- Value metrics for scrap sorting

**API Functions**:

### Grade Library Management
- `getAvailableGradeLibraries(methodId)` - Retrieve all grade libraries available for method
- `getActiveGradeLibraries(methodId)` - Retrieve loaded grade libraries for comparison
- `setLibraryActive(libraryId, active)` - Enable/disable specific grade library for matching
- `getActiveLibraries(methodId)` - Retrieve currently active libraries for grade matching
- `saveLibraryConfiguration(methodId)` - Persist active library selections

### Grade Matching
- `computeGradeMatch(composition, libraries)` - Calculate and rank match numbers, return best match
- `getGradeAlert(gradeId)` - Retrieve grade-specific alert or message from library

### Scrap Classification (Recycling Mode)
- `loadScrapLibraries()` - Load scrap-specific hierarchical grade libraries
- `setScrapConfidenceThresholds(thresholds)` - Configure relaxed thresholds for scrap sorting
- `classifyScrapType(composition)` - Match composition to scrap hierarchy
- `calculateScrapValue(scrapType, marketFactors)` - Compute value metrics
- `generateSortingRecommendation(scrapType, value, contamination)` - Bin assignment recommendations
- `updateDailyStatistics(scrapType, value)` - Increment running totals for facility management

**Dependencies**:
- Database (grade library storage)
- Calc Engine Manager (composition data input)
- Results Manager (result storage with grade match)

---

## 7. Results Manager

**Responsibility**: Manage test result storage, retrieval, deletion, and attachments (notes, images, GPS, spectrum). Complete result lifecycle.

**Key Characteristics**:
- Complete result persistence
- Date-based organization
- Attachment management (images, spectrum, notes, GPS)
- Result browsing and filtering

**API Functions**:

### Result Storage
- `saveTestResult(chemistry, notes, images, spectrum, timestamp)` - Store complete test result
- `storeTestResult(testIndex)` - Save individual test result (multi-test sequences)
- `saveAverageResult()` - Store final averaged result (multi-test)
- `saveScrapClassification(result, confidence, value)` - Store scrap classification result
- `saveComplianceResult(testId, passFailStatus)` - Store RoHS Pass/Fail determination
- `saveCalibrationCheckResult(deviations, status, timestamp)` - Store calibration check result

### Result Retrieval
- `getResultsByDateRange(year, month, day)` - Retrieve test results by date hierarchy
- `getResultsByDateRange(startDate, endDate)` - Retrieve filtered results for selection
- `getTestResultDetails(testId)` - Retrieve full test record for viewing
- `getTodaysResults()` - Retrieve all test results from current day

### Result Deletion
- `deleteTestResult(testId)` - Remove test result from storage

### Attachments & Metadata
- `attachNoteToTest(testId, noteText)` - Associate note with test result
- `attachImageToTest(testId, imageId)` - Associate captured image with test result
- `attachGPSToTest(testId, coordinates)` - Associate GPS coordinates with test result
- `storeCapturedImage(imageData)` - Save image to temporary storage
- `captureSpectrumSnapshot(testId)` - Save spectrum data with test result
- `getSavedReferenceSpectra()` - Retrieve saved reference spectra for comparison

**Dependencies**:
- Database (primary dependency for all storage/retrieval)
- Export Manager (provides result data for exports)

---

## 8. Export Manager

**Responsibility**: Configure and execute data exports. Manage export templates, destinations, and export operations (auto/manual/batch).

**Key Characteristics**:
- Export template management
- Multi-destination support (SD, USB, network, cloud)
- Auto-export and manual export
- Batch export operations

**API Functions**:

### Export Configuration
- `getExportTemplate()` - Retrieve current export template configuration
- `setExportContent(contentOptions)` - Configure what data to include in exports
- `setExportFormat(format)` - Set file format (CSV, PDF)
- `setFilenameConvention(pattern)` - Configure filename pattern/convention
- `setExportDestination(destinationType, path)` - Set destination (SD, USB, network folder)
- `validateExportDestination(destination)` - Verify destination is accessible and writable
- `saveExportConfiguration()` - Persist export settings

### Export Execution
- `enableAutoExport(enabled)` - Enable/disable automatic export after each test
- `exportTestResult(testId)` - Automatically export test result using configured template
- `exportResultBatch(resultIds)` - Export batch of results using template
- `logExportError(testId, error)` - Record export failure for troubleshooting

### Export Selection
- `selectResultsForExport(resultIds)` - Mark specific results for export

**Dependencies**:
- Results Manager (retrieves result data for export)
- Network Manager (for network/cloud destinations)
- Settings Manager (for export configuration persistence)
- Notification Manager (export completion notifications)

---

## 9. Network Manager

**Responsibility**: Manage network connectivity including network shares (CIFS/SMB), cloud pairing, and cloud synchronization. Network time sync.

**Key Characteristics**:
- Network share mounting
- Cloud authentication and sync
- Network connectivity monitoring
- Credential management (encrypted)

**API Functions**:

### Network Connectivity
- `checkNetworkConnectivity()` - Verify network connection is available

### Network Shares (CIFS/SMB)
- `getMountedNetworkShares()` - Retrieve currently mounted network shares
- `mountNetworkShare(serverAddress, sharePath, username, password)` - Mount CIFS/SMB share
- `validateNetworkShareAccess(shareId)` - Verify share is accessible and writable
- `unmountNetworkShare(shareId)` - Disconnect from network share
- `saveNetworkCredentials(shareId, credentials)` - Persist encrypted credentials

### Cloud Connectivity
- `initiateCloudPairing(pin)` - Begin cloud pairing process with PIN from portal
- `authenticateWithCloud(pin)` - Validate PIN and receive authentication tokens
- `saveCloudCredentials(tokens)` - Persist encrypted cloud authentication tokens
- `enableCloudSync(enabled)` - Enable/disable automatic cloud synchronization
- `checkCloudReachability()` - Verify cloud service is reachable before sync attempt
- `syncResultsToCloud(resultIds)` - Upload test results to cloud service
- `getCloudSyncStatus()` - Retrieve sync state and last sync timestamp

### Network Time Sync
- `enableNetworkTimeSync(enabled)` - Enable/disable automatic NTP synchronization
- `syncTimeFromNetwork()` - Trigger immediate time sync from NTP server
- `getTimeSyncStatus()` - Retrieve last sync timestamp and sync source

**Dependencies**:
- Network protocols (CIFS/SMB, HTTPS, NTP)
- Settings Manager (credential storage)
- Results Manager (cloud sync data source)
- Notification Manager (connection status notifications)

---

## 10. Settings Manager

**Responsibility**: Centralized configuration management for display settings, date/time, notes, and GPS. All user-configurable settings not specific to methods.

**Key Characteristics**:
- Settings persistence
- Configuration validation
- Display/UI configuration
- Notes template management

**API Functions**:

### Date/Time Settings
- `getCurrentDateTime()` - Retrieve current system date and time
- `setManualDateTime(datetime)` - Set date/time manually from user input

### Display & Language Settings
- `getDisplaySettings()` - Retrieve current display configuration
- `setBrightness(level)` - Adjust screen brightness (0-100%)
- `setScreenRotation(degrees)` - Set display rotation (0, 90, 180, 270)
- `setFontSize(size)` - Set font size (small/medium/large)
- `getAvailableLanguages()` - Retrieve list of installed language packs
- `setLanguage(languageCode)` - Set active language for UI
- `saveDisplaySettings()` - Persist display configuration

### Notes Configuration
- `getNoteTemplates()` - Retrieve available pre-set note templates
- `selectNoteTemplate(templateId)` - Choose a note template for current test
- `setCustomNoteText(text)` - Enter or edit custom note text
- `setNoteRequirement(timing, required)` - Configure if note is required (pre-test/post-test)
- `saveNoteConfiguration()` - Persist note settings and templates

### GPS Configuration
- `enableGPSTagging(enabled)` - Enable/disable GPS coordinate capture
- `saveGPSSettings()` - Persist GPS tagging configuration

**Dependencies**:
- Database (settings persistence)
- Notification Manager (settings change notifications)

---

## 11. Device Manager

**Responsibility**: Manage peripheral devices including Bluetooth, GPS hardware, and camera peripherals. Device pairing and peripheral operations.

**Key Characteristics**:
- Peripheral device control
- Bluetooth pairing management
- GPS hardware interface
- Printing operations

**API Functions**:

### Bluetooth & Printing
- `enableBluetooth(enabled)` - Turn Bluetooth radio on/off
- `getBluetoothStatus()` - Check if Bluetooth is enabled and operational
- `scanBluetoothDevices()` - Scan for nearby Bluetooth devices
- `pairBluetoothDevice(deviceId, pin)` - Pair with discovered Bluetooth device
- `getPairedDevices()` - Retrieve list of paired Bluetooth devices
- `unpairBluetoothDevice(deviceId)` - Remove Bluetooth device pairing
- `printTestResult(testId, printerId)` - Send test result to paired Bluetooth printer

### GPS Hardware
- `getGPSStatus()` - Check if GPS hardware is available and has satellite lock
- `getCurrentGPSCoordinates()` - Retrieve current latitude/longitude from GPS module

### Camera Information
- `getAttachedCameras()` - Retrieve list of connected camera hardware (aiming, panoramic)

**Dependencies**:
- Hardware drivers (Bluetooth, GPS)
- Settings Manager (device configuration persistence)
- Results Manager (for printing)
- Notification Manager (device status notifications)

---

## 12. Session Manager

**Responsibility**: User authentication, session lifecycle, and user preference management. Login/logout workflows.

**Key Characteristics**:
- User authentication
- Session state management
- User preferences loading
- Audit trail for sessions

**API Functions**:

### Authentication
- `authenticateUser(username, password)` - Validate user credentials against stored accounts
- `getUserProfile(userId)` - Retrieve user profile with preferences and permissions

### Session Lifecycle
- `initializeSession(userId)` - Create new session and set session context
- `loadUserPreferences(userId)` - Load user-specific settings (language, display, methods)
- `logSessionStart(userId, timestamp)` - Record session start for audit trail
- `logSessionEnd(userId, timestamp)` - Record session end for audit trail
- `clearSessionState()` - Clear current user session data and context
- `restoreUserSession()` - Restore active user session (wake from sleep)

### Data Persistence Prompts
- `checkUnsavedData()` - Verify if there are any unsaved changes or pending operations
- `promptSaveUnsavedData()` - Display save prompt if unsaved data exists
- `promptSaveData()` - Display save prompt if unsaved data exists (power management)

**Dependencies**:
- Database (user accounts, preferences, audit log)
- Settings Manager (user-specific settings)
- Notification Manager (session status notifications, navigation)
- Power Manager (session restore on wake)

---

## 13. Power Manager

**Responsibility**: System power state management including shutdown, sleep, and wake operations. Power state transitions and system state persistence.

**Key Characteristics**:
- Power state transitions
- System state save/restore
- Wake trigger detection
- Coordinated hardware power control

**API Functions**:

### Power State Detection & Transitions
- `detectWakeTrigger()` - Identify wake source (touch, button, timer, network, device)
- `executeShutdown()` - Complete system shutdown (cold boot required)
- `executeSleep()` - Enter low-power sleep mode (quick resume enabled)

### System State Management
- `saveSystemState()` - Persist current system state and configuration
- `restoreSystemState()` - Reload system state saved before sleep

### Display Power Control
- `powerOnDisplay()` - Turn on display and restore brightness settings
- `restorePreviousInterface()` - Return to interface state before sleep

**Dependencies**:
- Hardware Interface Manager (hardware power control)
- Safety & Interlock Manager (safety checks before shutdown)
- Session Manager (session state persistence)
- Test Execution Manager (verify no active test)
- Notification Manager (power state notifications)

---

## 14. Diagnostics Manager

**Responsibility**: Hardware diagnostics, system health monitoring, log management, and device information. Diagnostic data collection and log export.

**Key Characteristics**:
- Hardware health monitoring
- Multi-category diagnostics
- Log collection and export
- Version information

**API Functions**:

### Hardware Diagnostics
- `getDiagnosticCategories()` - Retrieve list of diagnostic categories (battery, system board, sensors)
- `getBatteryDiagnostics()` - Get battery voltage, charge level, health status
- `getSystemBoardDiagnostics()` - Get system board temperatures, voltages, component status
- `getHardwareComponentStatus(componentId)` - Get detailed status for specific hardware component
- `getSensorReadings()` - Get current sensor readings (temperature, pressure, etc.)
- `getErrorLogs(category)` - Retrieve error logs for specific diagnostic category

### Device Log Management
- `getLogCategories()` - Retrieve available log categories (System, Hardware, Safety, Error)
- `selectLogCategories(categoryIds)` - Mark specific log categories for export
- `setLogDateRange(startDate, endDate)` - Configure date range filter for log export
- `getAvailableLogDestinations()` - Retrieve available export destinations (USB, SD, Cloud)
- `selectLogExportDestination(destinationType)` - Set export destination
- `packageLogsForExport(categories, dateRange)` - Create compressed archive with selected logs
- `exportLogArchive(archiveFile, destination)` - Export log archive to selected destination
- `logExportAuditEvent(userId, categories, destination, timestamp)` - Record log export in audit trail

### Device Information
- `getFirmwareVersions()` - Get firmware versions for all hardware components
- `getSoftwareVersions()` - Get software versions (UI, middleware, database)
- `getBuildDates()` - Retrieve build dates for firmware and software components

### Calibration Events
- `getReferenceStandard()` - Retrieve reference sample specifications (expected composition, tolerances)
- `logCalibrationEvent(timestamp, status)` - Record calibration check in audit trail

**Dependencies**:
- Hardware Interface Manager (hardware status queries)
- Database (log storage, audit trail)
- Export Manager (log export operations)
- Notification Manager (diagnostic status notifications)

---

## 15. Update Manager

**Responsibility**: Firmware and software update management including update checking, downloading, installation, verification, and rollback. System backup and restart coordination.

**Key Characteristics**:
- Automatic and manual update checking
- Secure download and validation
- Staged installation (software then firmware)
- Automatic rollback on failure
- Audit logging

**API Functions**:

### Update Discovery
- `checkForUpdates()` - Query update server for available software and firmware updates
- `compareVersions(currentVersions, availableVersions)` - Determine which components have updates

### Pre-Update Validation
- `validatePreUpdateConditions()` - Check storage space, battery level, no active scans
- `createSystemBackup()` - Backup current system state before updates

### Update Download & Validation
- `downloadUpdate(updatePackage)` - Download update package from server
- `validateUpdateSignature(updatePackage)` - Verify digital signature and checksum

### Update Installation
- `installSoftwareUpdates(packages)` - Install UI, middleware, and database updates
- `installFirmwareUpdates(packages)` - Install firmware to X-ray controller, DPP, drivers
- `verifyUpdateInstallation(componentId)` - Confirm each update applied successfully
- `rollbackUpdate(componentId)` - Restore previous version if update fails

### Update Logging & Restart
- `logUpdateEvent(componentId, oldVersion, newVersion, status, timestamp)` - Record update in audit trail
- `scheduleSystemRestart()` - Restart system to complete update process

**Dependencies**:
- Network Manager (update server connectivity)
- Diagnostics Manager (version information, pre-update checks)
- Power Manager (system restart)
- Database (update audit log)
- Notification Manager (update notifications)

---

## 16. Compliance Manager

**Responsibility**: RoHS-specific compliance testing features including action level management, Pass/Fail classification, and compliance reporting.

**Key Characteristics**:
- Element-specific thresholds
- Automated Pass/Fail determination
- Compliance report generation
- Regulatory audit support

**API Functions**:

### RoHS Action Levels
- `getRoHSActionLevels(methodId)` - Retrieve configured action level thresholds for all RoHS elements
- `setElementActionLevel(methodId, element, threshold)` - Configure threshold value for specific element
- `saveRoHSConfiguration(methodId)` - Persist RoHS action level settings

### Compliance Evaluation
- `compareResultsToActionLevels(composition, actionLevels)` - Evaluate measured concentrations against thresholds
- `evaluateRoHSCompliance(composition, actionLevels)` - Determine Pass/Fail status for all elements
- `generateComplianceReport(complianceStatus)` - Create detailed compliance report with element-by-element status

**Dependencies**:
- Method Manager (RoHS method configuration)
- Calc Engine Manager (composition data)
- Results Manager (compliance result storage)
- Notification Manager (compliance notifications)

---

## 17. Notification Manager

**Responsibility**: Centralized UI notification and navigation management. All user-facing notifications, alerts, and UI navigation requests.

**Key Characteristics**:
- UI notification coordination
- Screen navigation
- User confirmation prompts
- Priority-based notification routing

**API Functions**:

### Test Notifications
- `notifyTestStarted()` - Send UI notification that test is running
- `notifyTestStopped()` - Send UI notification that test stopped
- `notifyResultsReady(gradeId, composition)` - Send UI notification with test results
- `notifySequenceComplete()` - Send UI notification with multi-test results
- `notifyAwaitingConfirmation()` - Prompt user for next test (multi-test sequences)
- `waitForUserConfirmation()` - Block until user approves (multi-test sequences)
- `notifyBatchLoaded(testCount)` - Send UI notification of loaded batch sequence

### Configuration Notifications
- `notifyMethodChanged(methodName)` - Send UI notification of active method change
- `notifyTimingUpdated(methodId)` - Send UI confirmation of timing change
- `notifyFactorSetChanged(factorSetName)` - Send UI notification of active factor set
- `notifyDisplayPreferencesChanged(methodId)` - Send UI notification to update result display
- `notifyTriggerLocked()` - Send UI notification and update Start button to lock icon

### Result Notifications
- `notifyResultDeleted(testId)` - Send UI notification of successful deletion
- `notifyScrapResults(scrapType, confidence, value, binRecommendation)` - Send scrap classification results
- `notifySpectrumUpdate(spectrumData, peaks)` - Send UI update with current spectrum and peak markers

### Export Notifications
- `notifyExportComplete(testId, destination)` - Send UI notification of successful export
- `notifyExportComplete(resultCount, destination)` - Send UI notification with export summary

### Network Notifications
- `notifyMountStatus(shareId, status)` - Send UI notification of network share mount status
- `notifyCloudPaired(accountInfo)` - Send UI notification of successful cloud pairing
- `notifySyncComplete(resultCount)` - Send UI notification of cloud sync completion

### Settings Notifications
- `notifyDateTimeChanged(datetime)` - Send UI notification of time change
- `notifyDisplaySettingsChanged()` - Send UI notification to apply new settings

### Device Notifications
- `notifyPrintComplete(testId)` - Send UI notification of successful print

### Session Notifications
- `notifySessionStarted(userName)` - Send UI notification of successful login
- `notifySessionEnded()` - Send UI notification of successful logout
- `navigateToMainInterface()` - Navigate UI to Live View or main screen
- `navigateToWelcomeScreen()` - Return UI to Welcome/Login screen

### Power Notifications
- `notifyWakeComplete()` - Send UI notification system is ready (wake from sleep)

### Safety Notifications
- `notifyInterlockStatus(status, failedInterlocks)` - Send UI notification of interlock state

### Compliance Notifications
- `notifyActionLevelExceeded(element, measuredValue, threshold)` - Send UI alert if RoHS threshold exceeded
- `notifyComplianceStatus(passFailStatus, violatingElements)` - Send UI notification with visual Pass/Fail indicators

### Diagnostic Notifications
- `notifyDiagnosticsReady(diagnosticData)` - Send UI notification with diagnostic information
- `notifyLogExportComplete(archiveFilename, destination)` - Send UI notification of log export
- `notifyAboutInfoReady(deviceInfo)` - Send UI notification with device information
- `notifyCalibrationStatus(status, deviations)` - Send UI notification of calibration check results

### Update Notifications
- `notifyUpdatesAvailable(updateList)` - Send UI notification with update options
- `notifyUpdateComplete(successfulUpdates, failedUpdates)` - Send UI notification with update results

### Adaptive Testing Notifications
- `notifyAdaptiveChange(changeType, newValues)` - Send UI update for adaptive parameter changes

**Dependencies**:
- ALL components (receives notifications from all)
- UI Layer (sends notifications to UI)

---

## Component Interaction Patterns

### Test Execution Flow
```
UI → Test Execution Manager
  ↓
  → Safety & Interlock Manager (validation)
  → Method Manager (get parameters)
  → Hardware Interface Manager (start hardware)
  → Calc Engine Manager (process spectrum)
  → Grade Library Manager (match grade)
  → Results Manager (save result)
  → Notification Manager (notify UI)
```

### Export Flow
```
UI → Export Manager
  → Results Manager (retrieve data)
  → Network Manager (validate destination)
  → Notification Manager (completion)
```

### Safety Event Flow
```
Hardware Interface Manager (LED failure)
  → Safety & Interlock Manager
    → Test Execution Manager (emergency stop)
    → Notification Manager (alert)
    → Diagnostics Manager (log event)
```

### Update Flow
```
Update Manager
  → Network Manager (download)
  → Diagnostics Manager (version check)
  → Power Manager (restart)
  → Notification Manager (user prompts)
```

---

## Cross-Cutting Concerns

### Database Integration
Components that persist data:
- Results Manager (primary)
- Method Manager
- Grade Library Manager
- Settings Manager
- Session Manager
- Export Manager
- Diagnostics Manager
- Compliance Manager

### Safety Integration
Components that must coordinate with Safety Manager:
- Test Execution Manager (primary coordinator)
- Hardware Interface Manager
- Method Manager (parameter validation)
- Power Manager (shutdown coordination)

### Hardware API Integration
Components with direct hardware dependencies:
- Hardware Interface Manager (primary)
- Safety & Interlock Manager (LED, interlocks)
- Diagnostics Manager (hardware status)
- Device Manager (peripherals)

---

## Implementation Considerations

### Thread Safety
Components requiring thread-safe operations:
- Test Execution Manager (real-time test monitoring)
- Hardware Interface Manager (async hardware operations)
- Calc Engine Manager (IPC with CalcEngine process)
- Notification Manager (async UI updates)

### Error Handling
- Each component responsible for own error handling
- Critical errors propagated through Notification Manager
- Safety errors handled immediately by Safety & Interlock Manager

### Logging
- All components use centralized logging (via Diagnostics Manager or common utility)
- Safety events logged for compliance
- All configuration changes logged
- Update events logged for audit

### State Management
- Components maintain own internal state
- State transitions coordinated through events
- System-wide state managed by Power Manager

---

## Component Dependencies Summary

| Component | Primary Dependencies |
|-----------|---------------------|
| Hardware Interface Manager | Hardware API, Safety Manager |
| Safety & Interlock Manager | Hardware Interface, Test Execution, Notification, Database |
| Test Execution Manager | Safety, Hardware Interface, Method, Calc Engine, Results, Notification |
| Method Manager | Database, Calc Engine, Notification |
| Calc Engine Manager | CalcEngine Process, Hardware Interface, Method |
| Grade Library Manager | Database, Calc Engine, Results |
| Results Manager | Database, Export |
| Export Manager | Results, Network, Settings, Notification |
| Network Manager | Network Protocols, Settings, Results, Notification |
| Settings Manager | Database, Notification |
| Device Manager | Hardware Drivers, Settings, Results, Notification |
| Session Manager | Database, Settings, Notification, Power |
| Power Manager | Hardware Interface, Safety, Session, Test Execution, Notification |
| Diagnostics Manager | Hardware Interface, Database, Export, Notification |
| Update Manager | Network, Diagnostics, Power, Database, Notification |
| Compliance Manager | Method, Calc Engine, Results, Notification |
| Notification Manager | ALL (receives from all components) |

---

## Notes

1. **Component Independence**: Each component has well-defined interfaces and minimal coupling to other components

2. **Notification Manager as Hub**: Centralized notification system prevents tight coupling between components and UI

3. **Safety First**: Safety & Interlock Manager has highest priority and can override any operation

4. **Extensibility**: New application modes (Mining, Jewelry) can be accommodated by extending existing components

5. **Testability**: Clear component boundaries enable unit testing and integration testing

6. **Scalability**: Components can be distributed across processes/threads as performance requires