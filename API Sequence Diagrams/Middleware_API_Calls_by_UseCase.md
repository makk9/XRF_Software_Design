# Middleware API Calls by Use Case

## Overview
This document lists the required API calls for each use case documented in Master Backend Usecases.md. API calls are listed in execution order and kept at a high level without component grouping (grouping will be done in Step 2).

---

## Measurement Control

### Use Case: Start/Stop Test

**Description**: The backend uses a single Start button in Live View to initiate an XRF test; once tapped, it becomes a Stop button until the test finishes. Tapping Start again during a running test stops it immediately.

**Required API Calls** (in order):

**Start Test Flow:**
1. `validateSafetyInterlocks()` - Check all safety conditions
2. `getActiveMethod()` - Retrieve currently selected method
3. `getTestTiming()` - Get configured min/max times
4. `initializeHardware()` - Initialize DPP and X-ray tube
5. `activateRadiationLED()` - Turn on radiation warning LED
6. `startXRayTube(voltage, current)` - Begin X-ray emission
7. `startSpectrumAcquisition()` - Begin DPP data collection
8. `notifyTestStarted()` - Send UI notification
9. `monitorTestExecution()` - Monitor timing and safety

**Stop Test Flow:**
1. `stopXRayTube()` - Cease X-ray emission
2. `deactivateRadiationLED()` - Turn off warning LED
3. `stopSpectrumAcquisition()` - Stop DPP collection
4. `notifyTestStopped()` - Send UI notification
5. `shutdownHardware()` - Power down components

---

### Use Case: Multi-Test Sequencing

**Description**: Users can configure the analyzer to repeat a test multiple times. In the Multiple Tests setup, the operator specifies a number of repeats and can optionally insert a prompt between tests (so the analyzer waits for confirmation before each successive run). The backend can automatically average the results of these repeats: a setting "calculate average" generates the mean composition of the series for display.

**Required API Calls** (in order):

1. `getMultiTestConfiguration()` - Retrieve repeat count and prompt settings
2. `executeTest()` - Run single test (calls Start/Stop Test flow internally)
3. `storeTestResult(testIndex)` - Save individual test result
4. `notifyAwaitingConfirmation()` - Prompt user for next test (if prompt enabled)
5. `waitForUserConfirmation()` - Block until user approves (if prompt enabled)
6. `calculateAverageResults(resultIds)` - Compute mean composition if enabled
7. `saveAverageResult()` - Store final averaged result
8. `notifySequenceComplete()` - Send UI notification with results

---

### Use Case: Safety Triggers

**Description**: The system enforces safety trigger modes. In Safety Settings, the user can require a deadman mode (operator must hold the trigger through the entire test), enable a timed trigger lock (automatically disables the trigger after an interval), or require two-handed operation (must hold the trigger and the back navigation button simultaneously). When a trigger lock is active, the Start button is disabled (replaced by a lock icon) and tests cannot be run until the trigger is explicitly unlocked via the System Tray.

**Required API Calls** (in order):

1. `getSafetyTriggerMode()` - Retrieve configured trigger mode (deadman/timed-lock/two-handed)
2. `validateTriggerConditions()` - Check if trigger requirements are met before allowing test
3. `monitorTriggerState()` - Continuously monitor trigger/button state during test (for deadman mode)
4. `enableTriggerLock()` - Activate trigger lock after timeout interval
5. `disableTriggerLock()` - Unlock trigger via System Tray action
6. `notifyTriggerLocked()` - Send UI notification and update Start button to lock icon
7. `abortTestOnTriggerRelease()` - Emergency stop if deadman trigger released during test

---

### Use Case: Radiation LED Safety Control

**Description**: The backend automatically controls the radiation warning LED in perfect synchronization with X-ray tube operation - this is a mandatory safety requirement. When a test starts and the X-ray tube activates, the radiation LED immediately turns ON to provide visual warning of X-ray emission. The LED remains illuminated throughout the entire test duration. When the test stops (either user-initiated or emergency stop), the LED automatically turns OFF as the X-ray tube shuts down. The backend continuously monitors LED circuit integrity during operation; if LED failure is detected, the system triggers an emergency safety response and immediately stops X-ray emission. All LED activation, deactivation, and fault events are logged with timestamps for regulatory compliance and audit trail requirements.

**Required API Calls** (in order):

1. `activateRadiationLED()` - Turn on radiation warning LED synchronized with X-ray tube start
2. `monitorLEDIntegrity()` - Continuously check LED circuit status during operation
3. `validateLEDTubeSync()` - Cross-check LED state matches X-ray tube state
4. `deactivateRadiationLED()` - Turn off radiation warning LED synchronized with X-ray tube stop
5. `triggerEmergencyStop()` - Immediately stop X-ray emission if LED failure detected
6. `logLEDEvent(eventType, timestamp)` - Record LED activation/deactivation/fault events for compliance

---

### Use Case: Workstation Interlock (Optional)

**SKIPPED** - Implementation details are internal and not user-configurable.

---

## Test Configuration

### Use Case: Method Selection

**Description**: The backend stores multiple calibrated analysis methods (Alloy, RoHS, Geochem, etc.). In Live View or the menu, the user taps "Select Method" to open the method list; only the methods that have been factory-calibrated (or loaded via PC software) appear. Selecting a method loads its parameters (e.g. which elements to report, pass/fail criteria).

**Required API Calls** (in order):

1. `getAvailableMethods()` - Retrieve list of factory-calibrated/loaded methods
2. `selectMethod(methodId)` - Set the active method for testing
3. `loadMethodParameters(methodId)` - Load method configuration (elements, criteria, beams)
4. `notifyMethodChanged(methodName)` - Send UI notification of active method change

---

### Use Case: Test Timing

**Description**: For each loaded method, the backend maintains minimum and maximum X-ray exposure times per beam. In the Test Times screen, the user sets Min and Max durations (in seconds) for each active beam. The actual test will run at least the Min time and up to Max; longer Max times improve statistical precision of the analysis. (For two-beam methods, each beam's times are set sequentially.)

**Required API Calls** (in order):

1. `getMethodBeamConfiguration(methodId)` - Retrieve number of beams and defaults for method
2. `setBeamTiming(beamId, minTime, maxTime)` - Configure and validate min/max times for beam
3. `saveTimingConfiguration(methodId)` - Persist timing settings for method
4. `notifyTimingUpdated(methodId)` - Send UI confirmation of timing change

---

### Use Case: Repeat/Batch Tests

**Description**: The user can choose Repeat Tests (repeat one sample multiple times) or run an imported Batch Test script. Repeat mode is configured in the Multiple Tests screen by entering a count and optional delay prompt. Batch Tests (pre-programmed sequences from PC) are similarly supported: the backend will execute each test in the sequence automatically once a batch is loaded.

**Required API Calls** (in order):

1. `setRepeatTestConfiguration(count, promptEnabled)` - Configure repeat test settings
2. `loadBatchTestScript(filePath)` - Import batch test sequence from PC-generated file
3. `getBatchTestSequence()` - Retrieve loaded batch test steps and parameters
4. `executeBatchSequence()` - Run batch test sequence automatically (calls executeTest() per step)
5. `saveBatchConfiguration()` - Persist batch/repeat settings
6. `notifyBatchLoaded(testCount)` - Send UI notification of loaded batch sequence

---

### Use Case: User Factors (Site Calibration)

**Description**: In Geochemistry methods, User Factors allow site-specific calibration. The backend lets the user create custom factor/offset tables to tweak the factory calibration for particular elements. Multiple named factor sets can be defined on-device: each is a list of multiplier/offset values per element. The user can switch among these factors on the fly without altering the factory calibration. (For each set, a linear correction is applied to the raw XRF result.)

**Required API Calls** (in order):

1. `getUserFactorSets(methodId)` - Retrieve all defined user factor sets for method
2. `createUserFactorSet(name, methodId)` - Create new named factor set
3. `setElementFactor(factorSetId, element, multiplier, offset)` - Configure factor/offset for specific element
4. `selectActiveFactorSet(factorSetId)` - Switch to different factor set for current method
5. `applyUserFactors(rawResults, factorSetId)` - Apply linear correction to raw XRF results
6. `saveUserFactorSet(factorSetId)` - Persist factor set configuration
7. `notifyFactorSetChanged(factorSetName)` - Send UI notification of active factor set

---

### Use Case: Method Display Preferences

**Description**: For each method, the user chooses which result columns to show. The Method Display screen provides checkboxes for items like Show LOD (elements below detection limit), Show Uncertainty (±), Show Chemistry Values, Show User Factor Name, Show Plating Alert, and Show Au Karat. For example, enabling "Show Uncertainty" adds ± columns to the elemental output; "Show Au Karat" converts gold content to karats on-screen.

**Required API Calls** (in order):

1. `getMethodDisplayOptions(methodId)` - Retrieve current display preferences for method
2. `setDisplayOption(methodId, optionName, enabled)` - Enable/disable specific display option
3. `saveDisplayPreferences(methodId)` - Persist display configuration for method
4. `notifyDisplayPreferencesChanged(methodId)` - Send UI notification to update result display

---

### Use Case: Custom Notes

**Description**: The backend can display pre-set text notes on the screen before or after a test. In Notes setup, the user selects from optional note templates or enters custom text. The notes screen is accessed via the Notes button and allows editing before or after running a test. The operator can also force entry of a note every test (pre-test or post-test) via a checkbox. The chosen note (and any typed text) is saved with the test result.

**Required API Calls** (in order):

1. `getNoteTemplates()` - Retrieve available pre-set note templates
2. `selectNoteTemplate(templateId)` - Choose a note template for current test
3. `setCustomNoteText(text)` - Enter or edit custom note text
4. `setNoteRequirement(timing, required)` - Configure if note is required (pre-test/post-test)
5. `attachNoteToTest(testId, noteText)` - Associate note with test result
6. `saveNoteConfiguration()` - Persist note settings and templates

---

### Use Case: Element Display Order

**SKIPPED** - Not directly configurable on-device via UI. Handled at factory or via PC-side method files.

---

### Use Case: Pseudo-Element Formulas

**SKIPPED** - Advanced feature configured via PC, not on-device.

---

### Use Case: Compound Calculations (GeoChem Methods)

**Description**: In Geochem methods, the backend can calculate compound mass fractions. The user adds compound templates (e.g. Fe→Fe₂O₃, Si→SiO₂) in the Compound setup. The analyzer then converts elemental wt% to oxide (or other compound) wt% using atomic weights. For example: if the sample contains 40 wt% Fe, the backend can display it as ~53.7 wt% Fe₂O₃, since Fe₂O₃ has a higher molecular weight than pure Fe.

**Required API Calls** (in order):

1. `getCompoundTemplates(methodId)` - Retrieve available compound conversion templates for method
2. `addCompoundTemplate(methodId, element, compound)` - Add new element-to-compound conversion
3. `calculateCompoundConversion(elementalResults)` - Convert elemental wt% to compound wt% using atomic weights
4. `enableCompoundDisplay(methodId, enabled)` - Toggle compound display on/off for method
5. `saveCompoundConfiguration(methodId)` - Persist compound calculation settings

---

### Use Case: Adaptive Time Testing

**Description**: The backend supports intelligent adaptive testing that dynamically optimizes measurement parameters based on real-time spectrum analysis. When adaptive mode is enabled in test configuration, the system continuously analyzes the live spectrum during measurement using the calculation engine (CalcEngine) to assess light element detection probability and measurement quality. Based on CalcEngine recommendations, the backend can automatically adjust X-ray tube parameters (voltage and current) during the test to optimize excitation for detected elements, particularly challenging light elements (low-Z). The system evaluates measurement confidence against configured thresholds and can extend test time dynamically (within configured maximum limits) when improved statistics are needed for reliable light element detection. Real-time UI feedback displays adaptive progress with dynamic time estimates, beam parameter changes, and light element detection probability indicators. The adaptive decision logic applies safety validation to all parameter changes, ensuring beam stability during transitions. All adaptive decisions, parameter modifications, and extensions are logged with CalcEngine recommendations for performance analysis and optimization.

**Required API Calls** (in order):

1. `enableAdaptiveMode(methodId, enabled)` - Enable/disable adaptive testing for method
2. `setAdaptiveConstraints(maxTime, maxVoltage, maxCurrent)` - Configure adaptive limits
3. `analyzeSpectrumQuality(liveSpectrum)` - Send live spectrum to CalcEngine for quality assessment
4. `getLightElementProbability()` - Retrieve CalcEngine detection probability for light elements
5. `adjustBeamParameters(voltage, current)` - Modify X-ray tube settings during test (with safety validation)
6. `extendTestTime(additionalSeconds)` - Dynamically extend test duration within max limits
7. `notifyAdaptiveChange(changeType, newValues)` - Send UI update for parameter changes
8. `logAdaptiveDecision(decision, calcEngineRecommendation)` - Record adaptive actions for analysis

---

## Results and Grade Matching

### Use Case: Composition and Grade Identification

**Description**: After each test, the backend processes the raw spectrum to compute elemental concentrations, then automatically compares this composition to the loaded grade libraries. In alloy mode, the Vanta checks each known grade's concentration ranges and computes a "match number." A match number of 0 indicates an exact specification match. The backend immediately displays the best-matched grade ID and the sample chemistry (usually within ~1 second). If multiple grades are possible (or none), it shows the list or a "no match" status.

**Required API Calls** (in order):

1. `retrieveFinalSpectrum()` - Get completed spectrum data from DPP
2. `calculateChemistry(spectrum)` - Process spectrum to compute elemental concentrations via CalcEngine
3. `getActiveGradeLibraries(methodId)` - Retrieve loaded grade libraries for comparison
4. `computeGradeMatch(composition, libraries)` - Calculate and rank match numbers, return best match
5. `notifyResultsReady(gradeId, composition)` - Send UI notification with results

---

### Use Case: Result Management

**Description**: Every test result (chemistry, notes, images, spectrum, etc.) is saved on the instrument with a timestamp. The Browse Results screen lets the user navigate by year/month/day and select individual test records. From here, results can be exported or deleted. Deleting requires a two-step tap ("tap Delete, then tap again while it turns red") to confirm. Each test is an independent record; there is no concept of partially saving test data.

**Required API Calls** (in order):

1. `saveTestResult(chemistry, notes, images, spectrum, timestamp)` - Store complete test result with all associated data
2. `getResultsByDateRange(year, month, day)` - Retrieve test results for specified date hierarchy
3. `getTestResultDetails(testId)` - Retrieve full test record for viewing
4. `deleteTestResult(testId)` - Remove test result from storage (requires confirmation)
5. `notifyResultDeleted(testId)` - Send UI notification of successful deletion

---

### Use Case: Scrap Type Classification (Recycling Mode)

**Description**: For recycling operations, the backend provides specialized scrap classification that leverages the existing grade matching engine with scrap-specific libraries and optimized settings. When operating in recycling mode, the system loads scrap-specific grade libraries organized in hierarchical classifications: Primary Categories (Ferrous, Non-Ferrous, Precious Metals, Specialty Alloys) → Alloy Families (Carbon Steel, Stainless, 6xxx Aluminum, etc.) → Specific Grades (304 SS, 6061 Al, C101 Cu, etc.) → Quality Levels (Clean, Contaminated, Mixed). The classification process uses the same match number calculation as standard grade matching but applies relaxed confidence thresholds optimized for high-throughput sorting. The backend calculates value metrics based on scrap type and facility-specific market factors, providing sorting recommendations and contamination warnings that affect material value. Results display the scrap type classification with confidence levels, value indicators, and bin assignment recommendations. The system maintains running totals and value summaries for facility management, updating daily statistics as each sample is classified. All scrap classification results are stored with confidence scores and value assessments for traceability and facility reporting.

**Required API Calls** (in order):

1. `loadScrapLibraries()` - Load scrap-specific hierarchical grade libraries for recycling mode
2. `setScrapConfidenceThresholds(thresholds)` - Configure relaxed confidence thresholds for high-throughput sorting
3. `calculateChemistry(spectrum)` - Process spectrum to compute elemental concentrations via CalcEngine
4. `classifyScrapType(composition)` - Match composition to scrap hierarchy with relaxed thresholds
5. `calculateScrapValue(scrapType, marketFactors)` - Compute value metrics based on type and facility factors
6. `generateSortingRecommendation(scrapType, value, contamination)` - Provide bin assignment and contamination warnings
7. `updateDailyStatistics(scrapType, value)` - Increment running totals for facility management
8. `saveScrapClassification(result, confidence, value)` - Store classification with traceability data
9. `notifyScrapResults(scrapType, confidence, value, binRecommendation)` - Send UI notification with classification results

---

### Use Case: Live Spectrum Display

**Description**: During active scans, the backend continuously streams real-time spectrum data from the DPP hardware through the middleware to the UI for live visualization. The spectrum display updates at regular intervals (typically every 1-2 seconds) throughout the measurement, showing the evolving X-ray spectrum as photon counts accumulate. The UI presents the spectrum as an interactive chart with energy (keV) on the X-axis and intensity (counts) on the Y-axis, allowing users to monitor data quality and peak development in real-time. The live spectrum view includes a spectrum carousel feature that allows users to swipe through multiple spectrum displays - the current live spectrum alongside any saved reference spectra for comparison. Peak markers automatically identify significant element peaks, and the display updates smoothly without blocking other UI operations. All spectrum data shown in the live view is captured and saved with the final test result for later review and analysis.

**Required API Calls** (in order):

1. `streamLiveSpectrum()` - Continuously stream real-time spectrum data from DPP to middleware
2. `identifyPeaks(liveSpectrum)` - Automatically detect and mark significant element peaks
3. `getSavedReferenceSpectra()` - Retrieve saved reference spectra for carousel comparison
4. `notifySpectrumUpdate(spectrumData, peaks)` - Send UI update with current spectrum and peak markers
5. `captureSpectrumSnapshot(testId)` - Save spectrum data with test result for later review

---

## Grade Library Management

### Use Case: Grade Library Management

**Description**: On-device, users can select which grade libraries are active for matching (e.g. ASTM steels, aluminum alloys). The backend only compares against grades in the checked libraries. (Custom libraries or cloned grades must be created/managed offline.) When a grade match occurs, the UI shows the grade name and any grade-specific alert or message as stored in the library.

**Required API Calls** (in order):

1. `getAvailableGradeLibraries(methodId)` - Retrieve all grade libraries available for method
2. `setLibraryActive(libraryId, active)` - Enable/disable specific grade library for matching
3. `getActiveLibraries(methodId)` - Retrieve currently active libraries for grade matching
4. `getGradeAlert(gradeId)` - Retrieve grade-specific alert or message from library
5. `saveLibraryConfiguration(methodId)` - Persist active library selections

---

## Data and Result Export

### Use Case: Export Configuration

**Description**: In Export Settings, the user defines export templates: choose content (chemistry values, uncertainties, sample images, spectrum data, etc.), file format (typically CSV or PDF), and filename conventions. The user also sets the export destination (microSD card, USB drive, or a mapped network folder) in this screen.

**Required API Calls** (in order):

1. `getExportTemplate()` - Retrieve current export template configuration
2. `setExportContent(contentOptions)` - Configure what data to include in exports (chemistry, uncertainties, images, spectrum)
3. `setExportFormat(format)` - Set file format (CSV, PDF)
4. `setFilenameConvention(pattern)` - Configure filename pattern/convention
5. `setExportDestination(destinationType, path)` - Set destination (microSD, USB, network folder)
6. `validateExportDestination(destination)` - Verify destination is accessible and writable
7. `saveExportConfiguration()` - Persist export settings

---

### Use Case: Auto-Export

**Description**: If Auto Export is enabled, the backend will automatically write each test result to the chosen destination immediately after the test completes. (By default it exports in CSV format.) This ensures every run is archived to the network/USB without manual action.

**Required API Calls** (in order):

1. `enableAutoExport(enabled)` - Enable/disable automatic export after each test
2. `getExportTemplate()` - Retrieve configured export settings (format, content, destination)
3. `exportTestResult(testId)` - Automatically export test result using configured template
4. `notifyExportComplete(testId, destination)` - Send UI notification of successful export
5. `logExportError(testId, error)` - Record export failure for troubleshooting

---

### Use Case: Manual Export (Export Today)

**Description**: Via the Export menu, the user can manually trigger "Export Today" to export all results from the current day. The backend packages today's tests using the configured export template and writes them to the selected destination (SD/USB/network).

**Required API Calls** (in order):

1. `getTodaysResults()` - Retrieve all test results from current day
2. `getExportTemplate()` - Retrieve configured export settings
3. `exportResultBatch(resultIds)` - Export batch of results using template
4. `notifyExportComplete(resultCount, destination)` - Send UI notification with export summary

---

### Use Case: Browse/Filter & Select (export subset)

**Description**: From Browse Results, the user can filter by date range, select specific test records (checkboxes), then tap Export Selected. The backend exports only the checked items to the configured destination using the active export template.

**Required API Calls** (in order):

1. `getResultsByDateRange(startDate, endDate)` - Retrieve filtered results for selection
2. `selectResultsForExport(resultIds)` - Mark specific results for export
3. `getExportTemplate()` - Retrieve configured export settings
4. `exportResultBatch(resultIds)` - Export selected results using template
5. `notifyExportComplete(resultCount, destination)` - Send UI notification with export summary

---

## Network and Cloud Connectivity

### Use Case: Network Folders (CIFS/SMB)

**Description**: The backend can mount Windows network shares (CIFS/SMB) as export targets. In Network Settings, the user enters the server address, share path, username, and password. Once mounted, the network folder appears as an available export destination. The backend manages connection state and credential storage.

**Required API Calls** (in order):

1. `getMountedNetworkShares()` - Retrieve currently mounted network shares
2. `mountNetworkShare(serverAddress, sharePath, username, password)` - Mount CIFS/SMB network share
3. `validateNetworkShareAccess(shareId)` - Verify share is accessible and writable
4. `unmountNetworkShare(shareId)` - Disconnect from network share
5. `saveNetworkCredentials(shareId, credentials)` - Persist encrypted credentials for auto-mount
6. `notifyMountStatus(shareId, status)` - Send UI notification of mount success/failure

---

### Use Case: Cloud Connectivity

**Description**: The backend supports pairing with Evident Cloud services for remote data synchronization and backup. In Cloud Settings, users initiate pairing by entering a PIN displayed on the cloud portal. Once paired, the backend can automatically sync test results to the cloud when network connectivity is available. The system maintains sync state and handles authentication tokens.

**Required API Calls** (in order):

1. `checkNetworkConnectivity()` - Verify network connection is available
2. `initiateCloudPairing(pin)` - Begin cloud pairing process with PIN from portal
3. `authenticateWithCloud(pin)` - Validate PIN and receive authentication tokens
4. `saveCloudCredentials(tokens)` - Persist encrypted cloud authentication tokens
5. `enableCloudSync(enabled)` - Enable/disable automatic cloud synchronization
6. `checkCloudReachability()` - Verify cloud service is reachable before sync attempt
7. `syncResultsToCloud(resultIds)` - Upload test results to cloud service
8. `getCloudSyncStatus()` - Retrieve sync state and last sync timestamp
9. `notifyCloudPaired(accountInfo)` - Send UI notification of successful pairing
10. `notifySyncComplete(resultCount)` - Send UI notification of sync completion

---

## DateTime & Display Settings

### Use Case: Date/Time Sync

**Description**: The backend can synchronize the instrument's date and time either from the network (NTP) or manually from user input. In Date/Time Settings, users can enable automatic network sync or manually enter date/time values. Accurate timestamps are critical for result traceability and audit compliance.

**Required API Calls** (in order):

1. `getCurrentDateTime()` - Retrieve current system date and time
2. `enableNetworkTimeSync(enabled)` - Enable/disable automatic NTP synchronization
3. `syncTimeFromNetwork()` - Trigger immediate time sync from NTP server
4. `setManualDateTime(datetime)` - Set date/time manually from user input
5. `getTimeSyncStatus()` - Retrieve last sync timestamp and sync source (network/manual)
6. `notifyDateTimeChanged(datetime)` - Send UI notification of time change

---

### Use Case: Display & Language Settings

**Description**: The backend manages display configuration including brightness, screen rotation, font size, and language selection. In Display Settings, users adjust brightness slider, choose rotation (0°, 90°, 180°, 270°), select font size (small/medium/large), and pick from available language packs. Settings take effect immediately and persist across reboots.

**Required API Calls** (in order):

1. `getDisplaySettings()` - Retrieve current display configuration
2. `setBrightness(level)` - Adjust screen brightness (0-100%)
3. `setScreenRotation(degrees)` - Set display rotation (0, 90, 180, 270)
4. `setFontSize(size)` - Set font size (small/medium/large)
5. `getAvailableLanguages()` - Retrieve list of installed language packs
6. `setLanguage(languageCode)` - Set active language for UI
7. `saveDisplaySettings()` - Persist display configuration
8. `notifyDisplaySettingsChanged()` - Send UI notification to apply new settings

---

## Camera & Imaging

### Use Case: GPS Tagging

**Description**: The backend can capture GPS coordinates and attach them to test results for location tracking. In GPS Settings, users enable/disable GPS tagging. When enabled, the backend retrieves current GPS coordinates from the device's GPS module and automatically embeds latitude/longitude data in each saved test result for traceability and mapping applications.

**Required API Calls** (in order):

1. `enableGPSTagging(enabled)` - Enable/disable GPS coordinate capture
2. `getGPSStatus()` - Check if GPS hardware is available and has satellite lock
3. `getCurrentGPSCoordinates()` - Retrieve current latitude/longitude from GPS module
4. `attachGPSToTest(testId, coordinates)` - Associate GPS coordinates with test result
5. `saveGPSSettings()` - Persist GPS tagging configuration

---

### Use Case: Bluetooth & Printing

**Description**: The backend manages Bluetooth connectivity for wireless peripherals, particularly printers. In Bluetooth Settings, users can enable/disable the Bluetooth radio, scan for nearby devices, and pair with compatible printers. Once paired, users can print test results directly from the device. The backend handles Bluetooth radio control, device discovery, pairing, and print job management.

**Required API Calls** (in order):

1. `enableBluetooth(enabled)` - Turn Bluetooth radio on/off
2. `getBluetoothStatus()` - Check if Bluetooth is enabled and operational
3. `scanBluetoothDevices()` - Scan for nearby Bluetooth devices
4. `pairBluetoothDevice(deviceId, pin)` - Pair with discovered Bluetooth device
5. `getPairedDevices()` - Retrieve list of paired Bluetooth devices
6. `unpairBluetoothDevice(deviceId)` - Remove Bluetooth device pairing
7. `printTestResult(testId, printerId)` - Send test result to paired Bluetooth printer
8. `notifyPrintComplete(testId)` - Send UI notification of successful print

---

## User Session Management

### Use Case: Session Login

**Description**: The backend manages user authentication and session initialization. At the Welcome screen, users enter credentials (username/password or PIN) to log in. The backend validates credentials, initializes user session with preferences and permissions, and navigates to the main application interface.

**Required API Calls** (in order):

1. `authenticateUser(username, password)` - Validate user credentials against stored accounts
2. `getUserProfile(userId)` - Retrieve user profile with preferences and permissions
3. `initializeSession(userId)` - Create new session and set session context
4. `loadUserPreferences(userId)` - Load user-specific settings (language, display, methods)
5. `logSessionStart(userId, timestamp)` - Record session start for audit trail
6. `navigateToMainInterface()` - Navigate UI to Live View or main screen
7. `notifySessionStarted(userName)` - Send UI notification of successful login

---

### Use Case: Session Logout

**Description**: The backend manages user session lifecycle. From the menu, users can tap "Logout" to end the current session and return to the Welcome/Login screen. The backend ensures any unsaved data is handled appropriately, clears session state, and navigates back to the login interface.

**Required API Calls** (in order):

1. `checkUnsavedData()` - Verify if there are any unsaved changes or pending operations
2. `promptSaveUnsavedData()` - Display save prompt if unsaved data exists
3. `clearSessionState()` - Clear current user session data and context
4. `logSessionEnd(userId, timestamp)` - Record session end for audit trail
5. `navigateToWelcomeScreen()` - Return UI to Welcome/Login screen
6. `notifySessionEnded()` - Send UI notification of successful logout

---

## Power Management

### Use Case: Shutdown/Sleep Modes

**Description**: The backend supports two power management modes accessible through the power menu. **Shutdown** completely powers down all system components, requiring a full cold boot on next startup with zero power consumption. **Sleep Mode** puts the analyzer into a low-power state while maintaining quick resume capability - the display powers off but the system remains responsive to wake triggers. Before initiating either mode, the backend performs safety checks: it prevents shutdown/sleep during active X-ray scans (displaying "Cannot shut down while scan in progress") and prompts to save any unsaved data. The shutdown sequence safely powers down the X-ray tube, DPP, and radiation LED in proper order, saves system state and configuration, then transitions to the selected power state.

**Required API Calls** (in order):

1. `checkActiveTest()` - Verify no X-ray scan is currently running
2. `checkUnsavedData()` - Check for unsaved test results or configuration changes
3. `promptSaveData()` - Display save prompt if unsaved data exists
4. `shutdownXRayTube()` - Safely power down X-ray tube
5. `shutdownDPP()` - Safely power down DPP hardware
6. `deactivateRadiationLED()` - Turn off radiation LED
7. `saveSystemState()` - Persist current system state and configuration
8. `executeShutdown()` - Complete system shutdown (cold boot required)
9. `executeSleep()` - Enter low-power sleep mode (quick resume enabled)

---

### Use Case: Wake from Sleep

**Description**: When in Sleep mode, the analyzer can be awakened by multiple triggers: touching the display, pressing hardware buttons, scheduled wake timers, or external events (network activity, device connection). The wake sequence immediately powers up essential components, restores the system state from sleep, turns on the display, and returns to the previous interface state. The system is ready for immediate use without requiring a full boot cycle.

**Required API Calls** (in order):

1. `detectWakeTrigger()` - Identify wake source (touch, button, timer, network, device)
2. `powerUpEssentialComponents()` - Restore power to CPU, display, essential hardware
3. `restoreSystemState()` - Reload system state saved before sleep
4. `powerOnDisplay()` - Turn on display and restore brightness settings
5. `restoreUserSession()` - Restore active user session if applicable
6. `restorePreviousInterface()` - Return to interface state before sleep
7. `notifyWakeComplete()` - Send UI notification system is ready

---

## RoHS Compliance Features

### Use Case: RoHS Action Levels

**Description**: In RoHS methods, the backend manages element-specific action levels (concentration thresholds) for regulated hazardous substances. Users can view and configure action levels for elements like Pb (lead), Cd (cadmium), Hg (mercury), Cr (chromium), Br (bromine), and Cl (chlorine). Each element has a threshold value (typically in ppm or wt%) that determines regulatory compliance. The backend compares measured concentrations against these thresholds during result evaluation.

**Required API Calls** (in order):

1. `getRoHSActionLevels(methodId)` - Retrieve configured action level thresholds for all RoHS elements
2. `setElementActionLevel(methodId, element, threshold)` - Configure threshold value for specific element
3. `compareResultsToActionLevels(composition, actionLevels)` - Evaluate measured concentrations against thresholds
4. `saveRoHSConfiguration(methodId)` - Persist RoHS action level settings
5. `notifyActionLevelExceeded(element, measuredValue, threshold)` - Send UI alert if threshold exceeded

---

### Use Case: RoHS Classification Mode

**Description**: The backend provides automated Pass/Fail classification for RoHS compliance testing. Based on the configured action levels, the system automatically evaluates each measured element concentration and determines overall compliance status. If any regulated element exceeds its action level threshold, the result is classified as "Fail" (non-compliant). If all elements are below thresholds, the result is "Pass" (compliant). The classification is displayed prominently with visual indicators (green for Pass, red for Fail) and saved with the test result for traceability.

**Required API Calls** (in order):

1. `getRoHSActionLevels(methodId)` - Retrieve action level thresholds for classification
2. `evaluateRoHSCompliance(composition, actionLevels)` - Determine Pass/Fail status for all elements
3. `generateComplianceReport(complianceStatus)` - Create detailed compliance report with element-by-element status
4. `saveComplianceResult(testId, passFailStatus)` - Store Pass/Fail determination with test result
5. `notifyComplianceStatus(passFailStatus, violatingElements)` - Send UI notification with visual Pass/Fail indicators

---

## Instrument Diagnostics & Safety Features

### Use Case: Hardware Diagnostics

**Description**: The backend includes a Diagnostics screen showing hardware status. Tapping "Diagnostics" in the menu opens a view of categories (battery, system board, etc.) and log files. Users can expand each category to view details (voltage levels, temperatures, error logs) to assess instrument health.

**Required API Calls** (in order):

1. `getDiagnosticCategories()` - Retrieve list of diagnostic categories (battery, system board, sensors, etc.)
2. `getBatteryDiagnostics()` - Get battery voltage, charge level, health status
3. `getSystemBoardDiagnostics()` - Get system board temperatures, voltages, component status
4. `getHardwareComponentStatus(componentId)` - Get detailed status for specific hardware component
5. `getSensorReadings()` - Get current sensor readings (temperature, pressure, etc.)
6. `getErrorLogs(category)` - Retrieve error logs for specific diagnostic category
7. `notifyDiagnosticsReady(diagnosticData)` - Send UI notification with diagnostic information

---

### Use Case: Device Log Export

**Description**: The backend provides comprehensive device log export capabilities accessible from the Diagnostics screen. Users can select specific **log categories** to export: System Logs (boot sequences, power events, configuration changes), Hardware Logs (DPP communication, X-ray tube control, sensor readings), Safety Logs (emergency stops, interlock violations, radiation LED events), and Error Logs (exceptions, failures, recovery actions). Each category can be individually selected or combined for export. The log export interface includes **filtering and date range selection** - users can specify start and end timestamps to export only relevant time periods, reducing file size and focusing on specific incidents or timeframes. The backend supports multiple **export destinations**: USB flash drive (when connected), SD card (if installed), or cloud storage (when network connectivity is available). During export, the system packages selected logs into compressed archive files with timestamps and device serial number in the filename for easy identification. All log exports are recorded in the audit trail with user ID, timestamp, categories exported, and destination for compliance tracking.

**Required API Calls** (in order):

1. `getLogCategories()` - Retrieve available log categories (System, Hardware, Safety, Error)
2. `selectLogCategories(categoryIds)` - Mark specific log categories for export
3. `setLogDateRange(startDate, endDate)` - Configure date range filter for log export
4. `getAvailableLogDestinations()` - Retrieve available export destinations (USB, SD, Cloud)
5. `selectLogExportDestination(destinationType)` - Set export destination
6. `packageLogsForExport(categories, dateRange)` - Create compressed archive with selected logs
7. `exportLogArchive(archiveFile, destination)` - Export log archive to selected destination
8. `logExportAuditEvent(userId, categories, destination, timestamp)` - Record log export in audit trail
9. `notifyLogExportComplete(archiveFilename, destination)` - Send UI notification of successful export

---

### Use Case: About Device

**Description**: An About screen lists instrument information: model number, serial number, which cameras are attached, firmware and software versions, and build dates. This is a read-only information screen accessed from the menu for support and troubleshooting purposes.

**Required API Calls** (in order):

1. `getAttachedCameras()` - Retrieve list of connected camera hardware (aiming, panoramic)
2. `getFirmwareVersions()` - Get firmware versions for all hardware components (X-ray controller, DPP, etc.)
3. `getSoftwareVersions()` - Get software versions (UI, middleware, database)
4. `getBuildDates()` - Retrieve build dates for firmware and software components
5. `notifyAboutInfoReady(deviceInfo)` - Send UI notification with device information

*Note: Model number and serial number are accessed directly from system configuration/constants initialized at boot, not via API calls.*

---

### Use Case: Calibration Check

**Description**: The backend supports periodic calibration verification using a reference sample (typically a 316 stainless steel coin). When the user initiates a calibration check from the Diagnostics menu, the system performs a test on the reference sample and compares the measured composition against known reference values. The backend calculates deviation from expected values and determines if the instrument is within calibration tolerance. If deviations exceed acceptable limits, the system alerts the user that recalibration may be needed. All calibration check results are logged with timestamps for compliance and audit purposes.

**Required API Calls** (in order):

1. `getReferenceStandard()` - Retrieve reference sample specifications (expected composition, tolerances)
2. `executeCalibrationTest()` - Perform test on reference sample (calls test execution flow)
3. `retrieveFinalSpectrum()` - Get completed spectrum from calibration test
4. `calculateChemistry(spectrum)` - Process spectrum to compute elemental concentrations
5. `saveCalibrationCheckResult(deviations, status, timestamp)` - Store calibration check result with pass/fail status
6. `notifyCalibrationStatus(status, deviations)` - Send UI notification of calibration check results
7. `logCalibrationEvent(timestamp, status)` - Record calibration check in audit trail

*Note: Comparison to reference values and status evaluation happen internally within the calibration component, not as separate API calls.*

---

### Use Case: Firmware and Software Updates

**Description**: The backend supports both automatic and manual system updates. **Automatic update checks** occur periodically (typically on user login), connecting to the update server to compare current versions against available updates. The system checks for both software updates (UI application, middleware, database schemas) and firmware updates (X-ray controller, DPP, hardware drivers). When updates are available, a notification banner appears with "Update Now" and "Later" options. **Manual update checks** can be initiated from the Settings menu via "Check for Updates" button, which immediately queries the update server. The update process includes security validation (digital signatures and checksums), system backup creation, and safe installation. Before installing, the backend verifies sufficient storage space, battery level, and ensures no active scans are running. During installation, software components update first, followed by firmware updates to hardware components if included. After completion, the system restarts and verifies all updates applied successfully. If any update fails, the backend automatically rolls back to the previous version to maintain system stability. All update events are logged with version numbers and timestamps for audit compliance.

**Required API Calls** (in order):

1. `checkForUpdates()` - Query update server for available software and firmware updates
2. `compareVersions(currentVersions, availableVersions)` - Determine which components have updates available
3. `notifyUpdatesAvailable(updateList)` - Send UI notification with update options
4. `validatePreUpdateConditions()` - Check storage space, battery level, no active scans
5. `createSystemBackup()` - Backup current system state before updates
6. `downloadUpdate(updatePackage)` - Download update package from server
7. `validateUpdateSignature(updatePackage)` - Verify digital signature and checksum
8. `installSoftwareUpdates(packages)` - Install UI, middleware, and database updates
9. `installFirmwareUpdates(packages)` - Install firmware to X-ray controller, DPP, drivers
10. `verifyUpdateInstallation(componentId)` - Confirm each update applied successfully
11. `rollbackUpdate(componentId)` - Restore previous version if update fails
12. `logUpdateEvent(componentId, oldVersion, newVersion, status, timestamp)` - Record update in audit trail
13. `scheduleSystemRestart()` - Restart system to complete update process
14. `notifyUpdateComplete(successfulUpdates, failedUpdates)` - Send UI notification with update results

---

### Use Case: Built-In Interlocks

**Description**: The backend enforces multiple built-in safety interlocks that prevent X-ray emission unless all safety conditions are met. These interlocks include: **Sample Present Detection** (verifies material is in contact with measurement window before allowing test), **Chamber Door Closed** (for workstation configurations, ensures chamber door is sealed), **Workstation Mode Verification** (confirms analyzer is properly docked in workstation if configured), and **Hardware Status Checks** (validates X-ray tube, DPP, and radiation LED are operational). All interlock states are continuously monitored during operation. If any interlock condition fails during an active test, the system immediately triggers an emergency stop. Interlock violations are logged with timestamps for safety auditing. The Start button remains disabled (grayed out or showing lock icon) until all interlock conditions are satisfied.

**Required API Calls** (in order):

1. `checkSamplePresent()` - Verify material is in contact with measurement window
2. `checkChamberDoorClosed()` - Verify chamber door is sealed (workstation mode)
3. `checkWorkstationDocked()` - Verify analyzer is properly docked (workstation mode)
4. `checkHardwareOperational()` - Validate X-ray tube, DPP, and LED are functional
5. `validateAllInterlocks()` - Aggregate check of all interlock conditions
6. `monitorInterlocksDuringTest()` - Continuously monitor interlocks during active test
7. `triggerEmergencyStop()` - Immediately stop X-ray emission if interlock fails
8. `logInterlockViolation(interlockType, timestamp)` - Record interlock failure for audit trail
9. `notifyInterlockStatus(status, failedInterlocks)` - Send UI notification of interlock state

---

### Use Case: Camera and Imaging

**Description**: The backend manages on-device camera hardware for sample documentation and targeting. The system supports multiple camera types: **Aiming Camera** (live view for precise sample positioning before test), **Panoramic Camera** (wide-angle context image showing sample and surroundings), and **Quick Camera Controls** (instant image capture with automatic attachment to test results). Users can access camera controls from Live View to capture images before or after tests. The **IR Collimator** control adjusts the laser targeting system for improved sample alignment. All captured images are automatically associated with test results and included in exports. The backend handles camera initialization, live preview streaming, image capture, storage, and device switching between different camera hardware.

**Required API Calls** (in order):

1. `initializeCamera(cameraType)` - Initialize specific camera hardware (aiming, panoramic)
2. `startCameraPreview(cameraType)` - Begin live camera preview stream to UI
3. `controlIRCollimator(enabled)` - Turn IR laser targeting on/off for sample alignment
4. `switchCamera(cameraType)` - Switch between aiming and panoramic cameras
5. `captureImage(cameraType)` - Capture still image from active camera
6. `storeCapturedImage(imageData)` - Save image to temporary storage
7. `attachImageToTest(testId, imageId)` - Associate captured image with test result
8. `stopCameraPreview()` - Stop camera preview and release camera hardware
9. `shutdownCamera()` - Power down camera hardware

---