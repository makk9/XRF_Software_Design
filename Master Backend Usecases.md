# Vanta Monarch Backend Use Cases (Updated)
## Measurement Control

### Start/Stop Test: 
The backend uses a single Start button in Live View to initiate an XRF test; once tapped, it becomes a Stop button until 
the test finishes. Tapping Start again during a running test stops it immediately.

### Multi-Test Sequencing: 
Users can configure the analyzer to repeat a test multiple times. In the Multiple Tests setup, the operator specifies 
a number of repeats and can optionally insert a prompt between tests (so the analyzer waits for confirmation before each
successive run). The backend can automatically average the results of these repeats: a setting “calculate average” 
generates the mean composition of the series for display.

### Safety Triggers:
The system enforces safety trigger modes. In Safety Settings, the user can require a deadman mode (operator must hold
the trigger through the entire test), enable a timed trigger lock (automatically disables the trigger after an interval),
or require two-handed operation (must hold the trigger and the back navigation button simultaneously). When a trigger
lock is active, the Start button is disabled (replaced by a lock icon) and tests cannot be run until the trigger is
explicitly unlocked via the System Tray.

**Radiation LED Safety Control:** The backend automatically controls the radiation warning LED in perfect
synchronization with X-ray tube operation - this is a mandatory safety requirement. When a test starts and the X-ray
tube activates, the radiation LED immediately turns ON to provide visual warning of X-ray emission. The LED remains
illuminated throughout the entire test duration. When the test stops (either user-initiated or emergency stop), the
LED automatically turns OFF as the X-ray tube shuts down. The backend continuously monitors LED circuit integrity
during operation; if LED failure is detected, the system triggers an emergency safety response and immediately stops
X-ray emission. All LED activation, deactivation, and fault events are logged with timestamps for regulatory compliance
and audit trail requirements.

### Workstation Interlock (Optional): 
In certain configurations (e.g. when using an optional Vanta Workstation for 50 kV mode), the backend can require 
that the analyzer be docked to its workstation before running high-voltage tests. (If the workstation accessory isn’t 
connected, the analyzer prevents tests at the higher tube settings.) [Note: Specific implementation details for the 
workstation interlock are handled internally and are not user-configurable.]

## Test Configuration

### Method Selection: 
The backend stores multiple calibrated analysis methods (Alloy, RoHS, Geochem, etc.). In Live View or the menu, the user 
taps “Select Method” to open the method list; only the methods that have been factory-calibrated 
(or loaded via PC software) appear. Selecting a method loads its parameters (e.g. which elements to 
report, pass/fail criteria).

### Test Timing: 
For each loaded method, the backend maintains minimum and maximum X-ray exposure times per beam. In the Test Times screen, 
the user sets Min and Max durations (in seconds) for each active beam. The actual test will run at least the Min time 
and up to Max; longer Max times improve statistical precision of the analysis. (For two-beam methods, each beam’s 
times are set sequentially.)

### Repeat/Batch Tests: 
The user can choose Repeat Tests (repeat one sample multiple times) or run an imported Batch Test script. Repeat mode is 
configured in the Multiple Tests screen by entering a count and optional delay prompt. Batch Tests (pre-programmed 
sequences from PC) are similarly supported: the backend will execute each test in the sequence automatically once a 
batch is loaded.

### User Factors (Site Calibration): 
In Geochemistry methods, User Factors allow site-specific calibration. The backend lets the user create custom 
factor/offset tables to tweak the factory calibration for particular elements. Multiple named factor sets can be 
defined on-device: each is a list of multiplier/offset values per element. The user can switch among these factors on 
the fly without altering the factory calibration.(For each set, a linear correction is applied to the raw XRF result.)

### Method Display Preferences: 
For each method, the user chooses which result columns to show. The Method Display screen provides checkboxes for items 
like Show LOD (elements below detection limit), Show Uncertainty (±), Show Chemistry Values, Show User Factor Name, 
Show Plating Alert, and Show Au Karat. For example, enabling “Show Uncertainty” adds ± columns to the elemental output; 
“Show Au Karat” converts gold content to karats on-screen.

### Custom Notes: 
The backend can display pre-set text notes on the screen before or after a test. In Notes setup, the user selects from 
optional note templates or enters custom text. The notes screen is accessed via the Notes button and allows editing 
before or after running a test. The operator can also force entry of a note every test (pre-test or post-test) via a 
checkbox. The chosen note (and any typed text) is saved with the test result.

### Element Display Order: 
(Not directly configurable on-device via UI.) The Vanta’s result display follows a fixed sequence of elements for each 
method. (Custom ordering is handled at the factory or via PC-side method files.)

### Pseudo-Element Formulas: 
(Advanced feature via PC.) The analyzer can compute “pseudo-elements” from measured values (for example, element ratios). 
Users define formulas on PC (e.g. Ceq = C + Si/6 + Mn/6 for carbon equivalents). The backend then applies these formulas 
to the measured elements and shows the pseudo-element in the results.

### Compound Calculations (GeoChem Methods): 
In Geochem methods, the backend can calculate compound mass fractions. The user adds compound templates 
(e.g. Fe→Fe₂O₃, Si→SiO₂) in the Compound setup. The analyzer then converts elemental wt% to oxide (or other compound) 
wt% using atomic weights. For example: if the sample contains 40 wt% Fe, the backend can display it as 
~53.7 wt% Fe₂O₃, since Fe₂O₃ has a higher molecular weight than pure Fe.

### Adaptive Time Testing:
The backend supports intelligent adaptive testing that dynamically optimizes measurement parameters based on
real-time spectrum analysis. When adaptive mode is enabled in test configuration, the system continuously analyzes
the live spectrum during measurement using the calculation engine (CalcEngine) to assess light element detection
probability and measurement quality. Based on CalcEngine recommendations, the backend can automatically adjust
X-ray tube parameters (voltage and current) during the test to optimize excitation for detected elements,
particularly challenging light elements (low-Z).

The system evaluates measurement confidence against configured thresholds and can **extend test time** dynamically
(within configured maximum limits) when improved statistics are needed for reliable light element detection.
Real-time UI feedback displays adaptive progress with dynamic time estimates, beam parameter changes, and
light element detection probability indicators. The adaptive decision logic applies safety validation to all
parameter changes, ensuring beam stability during transitions. All adaptive decisions, parameter modifications,
and extensions are logged with CalcEngine recommendations for performance analysis and optimization. This feature
is particularly valuable for applications requiring robust detection of elements like Mg, Al, Si, and P.

## Results and Grade Matching

### Composition and Grade Identification:
After each test, the backend processes the raw spectrum to compute elemental concentrations, then automatically compares 
this composition to the loaded grade libraries. In alloy mode, the Vanta checks each known grade’s concentration ranges 
and computes a “match number.” A match number of 0 indicates an exact specification match. The backend immediately 
displays the best-matched grade ID and the sample chemistry (usually within ~1 second). If multiple grades are possible 
(or none), it shows the list or a “no match” status.

### Result Management:
Every test result (chemistry, notes, images, spectrum, etc.) is saved on the instrument with a timestamp. The Browse
Results screen lets the user navigate by year/month/day and select individual test records. From here, results can be
exported or deleted. Deleting requires a two-step tap ("tap Delete, then tap again while it turns red") to confirm.
Each test is an independent record; there is no concept of partially saving test data.

### Scrap Type Classification (Recycling Mode):
For recycling operations, the backend provides specialized scrap classification that leverages the existing grade
matching engine with scrap-specific libraries and optimized settings. When operating in recycling mode, the system
loads scrap-specific grade libraries organized in hierarchical classifications: **Primary Categories** (Ferrous,
Non-Ferrous, Precious Metals, Specialty Alloys) → **Alloy Families** (Carbon Steel, Stainless, 6xxx Aluminum, etc.)
→ **Specific Grades** (304 SS, 6061 Al, C101 Cu, etc.) → **Quality Levels** (Clean, Contaminated, Mixed).

The classification process uses the same match number calculation as standard grade matching but applies
relaxed confidence thresholds optimized for high-throughput sorting. The backend calculates **value metrics**
based on scrap type and facility-specific market factors, providing sorting recommendations and contamination
warnings that affect material value. Results display the scrap type classification with confidence levels,
value indicators, and bin assignment recommendations. The system maintains running totals and value summaries
for facility management, updating daily statistics as each sample is classified. All scrap classification results
are stored with confidence scores and value assessments for traceability and facility reporting.

### Live Spectrum Display:
During active scans, the backend continuously streams real-time spectrum data from the DPP hardware through the
middleware to the UI for live visualization. The spectrum display updates at regular intervals (typically every
1-2 seconds) throughout the measurement, showing the evolving X-ray spectrum as photon counts accumulate. The UI
presents the spectrum as an interactive chart with energy (keV) on the X-axis and intensity (counts) on the Y-axis,
allowing users to monitor data quality and peak development in real-time.

The live spectrum view includes a **spectrum carousel** feature that allows users to swipe through multiple
spectrum displays - the current live spectrum alongside any saved reference spectra for comparison. Peak markers
automatically identify significant element peaks, and the display updates smoothly without blocking other UI
operations. All spectrum data shown in the live view is captured and saved with the final test result for later
review and analysis.

## Grade Library Management

On-device, users can select which grade libraries are active for matching (e.g. ASTM steels, aluminum alloys).
The backend only compares against grades in the checked libraries. (Custom libraries or cloned grades must be 
created/managed offline.) When a grade match occurs, the UI shows the grade name and any grade-specific alert or 
message as stored in the library.

## Data and Result Export

### Export Configuration: 
In Export Settings, the user defines export templates: choose content (chemistry values, uncertainties, sample images, 
spectrum data, etc.), file format (typically CSV or PDF), and filename conventions. The user also sets the export 
destination (microSD card, USB drive, or a mapped network folder) in this screen.

### Auto-Export: 
If Auto Export is enabled, the backend will automatically write each test result to the chosen destination immediately 
after the test completes. (By default it exports in CSV format.) This ensures every run is archived to the network/USB 
without manual action.

### Manual Export (Export Today): 
There is an Export Today action that bundles all results from the current day into one output file. Tapping Export Today 
in the menu (or via System Tray) writes a CSV of that day’s tests to the configured destination. A valid export location 
must already be set up in Export Settings.

### Browse/Filter & Select: 
In Browse Results, users can select any subset of stored results (by date or individual checkboxes). The Export action 
will then output only those selected tests. The smallest selectable unit is one test. After selection, tapping the 
Export button writes the chosen records (respecting the export template) to the chosen destination.

### Network Folders: 
The backend can connect to network shares. Under a Network Folder screen, the user adds or mounts CIFS/SMB shares by 
entering credentials. Only mounted folders appear as destinations for export. Once mounted, results can be 
auto-exported or manually exported to those network locations.

### Cloud Connectivity: 
The Vanta can link to the Evident Cloud for data sync. The user must first register the instrument on the Evident 
web portal to get a PIN. Entering that PIN in Cloud Settings on the device pairs it with the cloud account. 
After pairing, test data and configurations can be shared with the cloud service. 
(Detailed cloud behavior depends on account setup; cloud use is optional.)

### Date/Time Sync: 
The backend maintains its system clock. In Date & Time settings, the user can turn on “Automatic date & time” to sync 
via the network, or set time zone and format manually. Changing the 12h/24h format or time zone updates all 
timestamps shown in UI and in exported data.

### Display & Language Settings: 
In Display Settings, the user can adjust font size, screen rotation (enable/disable auto-rotate), and backlight brightness. 
There is also a quick Brightness toggle in the System Tray for Low/Medium/High modes. The UI supports multiple 
languages: tapping the Language option opens a list, and selecting a language immediately switches all on-screen text to 
that language.

### GPS Tagging: 
If equipped with GPS, toggling GPS on in Settings causes the analyzer to tag each test result with coordinates. 
When GPS is active, Live View shows the current latitude/longitude on-screen, and the exported data includes a location field. 
Users enable GPS in the menu or via a quick-action button, and can toggle it on/off as needed.

### Bluetooth & Printing: 
The backend’s Bluetooth radio can be turned on/off (either in the System Tray or via Bluetooth Settings). When on, the 
Vanta can pair with supported devices. In practice this is used to connect to an approved Bluetooth printer 
(only Zebra printers are supported by default). Once paired, the Print action in Browse Results will send the 
current screen or selected results to the printer. The Print button (paper icon) invokes Bluetooth printing according 
to the configured printer profile.

## Camera and Imaging

### Aiming Camera: 
The Vanta supports an optional aiming camera with a collimator. In the Camera settings, the user can enable the aiming 
camera and its built-in IR collimator. When enabled, swiping left from Live View shows the aiming camera view with a red 
targeting circle. The user places the sample under the beam so it appears in the circle. Tapping and holding the red 
“shutter” circle focuses (shrinks the circle to indicate focus), and then tapping Start will also record a snapshot 
through the aiming camera. That aiming-photo is automatically saved with the test result.

### Panoramic Camera: 
The panoramic (sample) camera is always active in Live View. By default Live View shows the panoramic view; to switch 
to aiming view one swipes or taps the Switch Camera button. In panoramic mode, tapping the Take Picture icon captures 
a photo of the sample. Each captured image thumbnail appears at the bottom; these images are not immediately saved as 
a separate file but are stored with the upcoming test. When the user later taps Start on a test, all pending panoramic 
images taken are written into that test’s result. Multiple panoramic snapshots can be taken per test 
(just tap Take Picture repeatedly). After the test, the user can view saved images by tapping the image (“+”) bar and 
swiping through the thumbnails.

### Quick Camera Controls: 
To quickly toggle the aiming camera without swiping screens, the aiming camera action button in the System Tray can turn 
it on/off. (There is no quick toggle for the panoramic camera—it simply remains active when not in aiming mode.) 
All saved camera images are bundled with the test result and can be exported or printed along with the chemistry.

## User Session

### Session Logout:
The backend provides a Logout Session action (in the Menu Tray). Tapping "Logout Session" immediately ends the
current user session and clears any session state, returning to the welcome screen. This forces the next user to re-start
a session from the welcome screen. (The Vanta does not implement separate user login accounts or roles beyond this
session-level logout.)

## Power Management

### Shutdown/Sleep Modes:
The backend supports two power management modes accessible through the power menu. **Shutdown** completely powers
down all system components, requiring a full cold boot on next startup with zero power consumption. **Sleep Mode**
puts the analyzer into a low-power state while maintaining quick resume capability - the display powers off but
the system remains responsive to wake triggers. Before initiating either mode, the backend performs safety checks:
it prevents shutdown/sleep during active X-ray scans (displaying "Cannot shut down while scan in progress") and
prompts to save any unsaved data. The shutdown sequence safely powers down the X-ray tube, DPP, and radiation LED
in proper order, saves system state and configuration, then transitions to the selected power state.

### Wake from Sleep:
When in Sleep mode, the analyzer can be awakened by multiple triggers: touching the display, pressing hardware
buttons, scheduled wake timers, or external events (network activity, device connection). The wake sequence
immediately powers up essential components, restores the system state from sleep, turns on the display, and
returns to the previous interface state. The system is ready for immediate use without requiring a full boot cycle.

## Regulatory Test Settings (RoHS)

### RoHS Action Levels: 
In RoHS methods, the backend allows adjusting statistical pass/fail criteria. In RoHS Action Level settings, 
the user can set a custom nSigma multiplier and enter pass/fail cutoff percentages for each classification 
(Alloy, Mixed, Plastic). The nSigma value effectively widens or narrows the action level boundaries 
(pass cutoff + nSigma·σ). For example, raising nSigma makes the analyzer more lenient (sample must be further above 
threshold to fail). The standard default EAC (EU/Asia) limits are always visible but can be overridden by entering new values.

### RoHS Classification Mode: 
For the automatic alloy/plastic classification, the user can force a specific mode. In the Test Times screen for RoHS, 
a “Force Classification” option offers Auto, Forced Plastic, or Forced Alloy. “Auto” lets the analyzer decide the 
material type normally. “Forced Plastic” forces use of the plastic (polymer) calibration even if the sample might be 
metal. “Forced Alloy” forces the metal calibration (useful for Al alloys that might otherwise be misclassified). This 
ensures the correct reference calibration is applied for borderline samples.

## Device Connectivity & Configuration

### Display Options: 
As described above, the Display menu stores screen preferences (brightness, font size, rotation). Language selection is 
also here: tapping “Language” opens the list of available UI languages, and choosing one immediately updates the 
interface text.

## Instrument Diagnostics & Safety Features

### Hardware Diagnostics:
The backend includes a Diagnostics screen showing hardware status. Tapping "Diagnostics" in the menu opens a view of
categories (battery, system board, etc.) and log files. Users can expand each category to view details (voltage levels,
temperatures, error logs) to assess instrument health.

### Device Log Export:
The backend provides comprehensive device log export capabilities accessible from the Diagnostics screen. Users can
select specific **log categories** to export: System Logs (boot sequences, power events, configuration changes),
Hardware Logs (DPP communication, X-ray tube control, sensor readings), Safety Logs (emergency stops, interlock
violations, radiation LED events), and Error Logs (exceptions, failures, recovery actions). Each category can be
individually selected or combined for export.

The log export interface includes **filtering and date range selection** - users can specify start and end timestamps
to export only relevant time periods, reducing file size and focusing on specific incidents or timeframes. The backend
supports multiple **export destinations**: USB flash drive (when connected), SD card (if installed), or cloud storage
(when network connectivity is available). During export, the system packages selected logs into compressed archive
files with timestamps and device serial number in the filename for easy identification. All log exports are recorded
in the audit trail with user ID, timestamp, categories exported, and destination for compliance tracking.

### About Device: An About screen lists instrument information: model number, serial number, which cameras are attached, 
and current software/firmware versions. It also shows regulatory and license info. This helps verify the instrument’s 
identity and update status.

### Calibration Check: 
The Cal Check action (in the menu) performs an internal calibration verification. When pressed, the Vanta takes a 
quick measurement of the built-in 316 stainless steel reference coin. It then reports “Pass” or “Fail” based on whether 
the result is within tolerance. This ensures the instrument remains calibrated between formal calibrations.

### Firmware and Software Updates:
The backend supports both automatic and manual system updates. **Automatic update checks** occur periodically
(typically on user login), connecting to the update server to compare current versions against available updates.
The system checks for both software updates (UI application, middleware, database schemas) and firmware updates
(X-ray controller, DPP, hardware drivers). When updates are available, a notification banner appears with
"Update Now" and "Later" options. **Manual update checks** can be initiated from the Settings menu via
"Check for Updates" button, which immediately queries the update server.

The update process includes security validation (digital signatures and checksums), system backup creation,
and safe installation. Before installing, the backend verifies sufficient storage space, battery level, and
ensures no active scans are running. During installation, software components update first, followed by firmware
updates to hardware components if included. After completion, the system restarts and verifies all updates
applied successfully. If any update fails, the backend automatically rolls back to the previous version to
maintain system stability. All update events are logged with version numbers and timestamps for audit compliance.

### Built-In Interlocks:
Beyond triggers, the backend enforces any required safety interlocks (e.g. two-hand operation for certain regions) as per the configuration. If a safety condition is violated (for example, no sample present or chamber door open when X-rays would fire), the system automatically aborts the test. Two-handed trigger and other interlock logic are pre-configured by Olympus/Evident and are not user-modifiable.