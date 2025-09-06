# XRF Analyzer System - Application Specific Use Cases

## Use Case 1: Grade Match ** NOT SURE 

### Overview
Automated grade identification system that compares measured chemistry against grade library specifications to determine material grade matches. System calculates match numbers, applies statistical confidence boundaries, and provides grade determination results for PMI applications.

### Sequence Flow

```
Actor: System (Continuous Process)
UI Layer: User Interface
MW API: Middleware API
DB: Database
```

**Step-by-Step API Flow:**

### Grade Match Configuration

1. **Grade Library Selection**
   ```
   User -> UI: Select grade match from menu
   UI -> MW API: GET /api/grade-match/libraries (async)
   MW API -> DB: getGradeLibraries(current_test_mode)
   DB -> MW API: Return available_libraries
   MW API -> UI: Return library_options
   UI: Display grade library selection interface
   
   User -> UI: Select a grade match library from selection interface
   UI -> MW API: POST /api/grade-match/select (async)
   MW API: Load selected grade match library details into current_grade_match state
   MW API -> UI: Display selected grade match library onto screen
   ```

2. **Grade Match Settings Configuration**
   ```
   User -> UI: Configure nSigma value, match cutoff, display options
   User -> UI: Set confidence thresholds and alert settings
   UI -> MW API: POST /api/grade-match/configure (settings, confidence_params) (async)
   MW API: Store grade match configuration and confidence parameters
   MW API: Validate settings against current test mode (PMI/reCycling)
   MW API: Set confidence thresholds for different application modes
   MW API -> UI: Return configuration_status
   UI: Display nSigma confidence settings in Grade Match configuration screen
   ```

### Live Grade Matching Process

3. **Chemistry Data Reception** (From Live Chemistry Use Case)
   ```
   MW API: Receive new chemistry data from live analysis
   MW API: Extract elemental concentrations and uncertainties
   MW API: Check if grade matching is enabled for current mode
   ```

4. **Grade Library Loading**
   ```
   MW API -> DB: loadGradeSpecs(selected_library, test_mode)
   DB: Query grade specifications for selected library
   DB: Filter grades by base material type and test mode
   DB -> MW API: Return grade_specifications (elements, ranges, tolerances)
   ```

5. **Match Number Calculation with Confidence Processing**
   ```
   MW API: Compare measured chemistry to each grade specification
   MW API: Calculate nSigma boundaries (grade_range ± nSigma × uncertainty)
   MW API: Process measurement precision data to determine confidence intervals
   MW API: Calculate confidence levels based on nSigma and measurement uncertainty
   MW API: Apply confidence weighting to grade match calculations
   MW API: Calculate match number for each grade
   MW API: Formula: match_number = Σ|measured_value - grade_nominal|/grade_tolerance
   MW API: Generate confidence scores for grade match reliability
   MW API: Determine statistical confidence boundaries for each grade specification
   MW API: Generate match_results (grade_id, match_number, confidence_level, uncertainty_ranges)
   ```

6. **Match Filtering and Ranking**
   ```
   MW API: Filter results by "Show Match No <" cutoff value
   MW API: Sort grades by match number (lower = better match)
   MW API: Apply grade separation logic for multiple matches
   MW API: Determine match status (exact_match/multiple_matches/no_match)
   ```

7. **Residual Element Processing** (if enabled)
   ```
   MW API: Process residual (tramp) elements by base material type
   MW API: Check for elements outside grade specifications
   MW API: Flag residual elements that exceed thresholds
   MW API: Include residual analysis in match determination
   ```

8. **Grade Match Results Generation**
   ```
   MW API: Generate final grade match results
   MW API: Include best match grade, confidence level, match number
   MW API: Generate confidence-based color coding for grade match quality
   MW API: Create statistical boundary visualizations indicating confidence ranges
   MW API: Generate color-coding for concentration bars based on confidence
   MW API: Prepare grade message and nominal chemistry display with uncertainty values
   MW API: Apply confidence thresholds for different application modes (PMI vs scrap sorting)
   ```

9. **Live Results Update with Confidence Display**
   ```
   MW API -> UI: updateGradeMatchResults(match_data, confidence, uncertainty_ranges, display_elements) (async)
   UI: Display grade match results in Live View
   UI: Show match numbers and confidence indicators alongside grade match results
   UI: Present measurement uncertainty values (± readings) with elemental results
   UI: Display confidence-based color coding for grade match quality
   UI: Show statistical boundary visualizations indicating confidence ranges
   UI: Display color-coded concentration bars (green=within confidence range, yellow=marginal, red=outside boundaries)
   UI: Present match determination (exact/multiple/no match) with confidence level indicators
   UI: Provide confidence level indicators for each matched grade
   UI: Show grade message and nominal chemistry with uncertainty bounds
   ```

10. **Match Results Storage**
    ```
    MW API -> DB: saveGradeMatchResults(session_id, match_results, confidence_level, uncertainty_data)
    MW API -> DB: saveConfidenceParameters(method_id, nSigma_settings, confidence_thresholds)
    ```

### API Sequence Diagram

```
System   MW API            DB                UI
 |        |                 |                 |
 |        | [chemistry received]             |  1. New chemistry from live analysis
 |        |---------------->|                |  2. loadGradeSpecs()
 |        |<----------------|                |  3. Return grade_specifications
 |        | [calc confidence] |              |  4. Process measurement precision & confidence
 |        | [calc matches]  |                |  5. Calculate match numbers vs grades with confidence
 |        | [filter/rank]   |                |  6. Filter by cutoff & rank by match#
 |        | [residuals]     |                |  7. Process residual elements
 |        | [confidence boundaries] |        |  8. Generate confidence-based visualizations
 |        |--------------------------------->|  9. updateGradeMatchResults() with confidence
 |        |                 |                | [display] 10. Show grade match & confidence results
 |        |---------------->|                | 11. saveGradeMatchResults() with confidence data
 |        |                 |                |
```

### Grade Match Determination Logic
- **Exact Match**: Single grade with match number below threshold
- **Multiple Matches**: Multiple grades within acceptable match range
- **No Match**: All grades exceed maximum match number threshold
- **Best Match Mode**: Always show best matching grade regardless of threshold

### Grade Comparison Modes
- **Best Match**: Display the lowest match number grade
- **Pass/Fail Grade**: Compare against specific reference grade
- **Selected Grade**: User-selected grade for comparison
- **None**: No grade comparison, chemistry only

### Grade Library Types
- **Factory Libraries**: Pre-installed industry standard grades (ASTM, AISI, etc.)
- **User Libraries**: Custom user-defined grades and specifications
- **Residuals Libraries**: Tramp element specifications by base material

### Confidence Management System
- **Configuration**: nSigma confidence settings in Grade Match configuration screen
- **Real-Time Display**: Confidence indicators alongside grade match results in Live View
- **Uncertainty Visualization**: Measurement uncertainty values (± readings) with elemental results
- **Color Coding**: Confidence-based color coding for grade match quality visualization
- **Statistical Boundaries**: Boundary visualizations indicating confidence ranges
- **Application-Specific**: Different confidence thresholds for PMI vs scrap sorting applications

### Statistical Confidence Calculation
- **nSigma Boundaries**: Grade_range ± (nSigma × measurement_uncertainty)
- **Confidence Levels**: High (>95%), Medium (90-95%), Low (<90%)
- **Uncertainty Propagation**: Combines measurement and grade specification uncertainties
- **Precision Processing**: Uses measurement precision data to determine confidence intervals
- **Weighting**: Confidence weighting applied to grade match calculations
- **Reliability Scoring**: Confidence scores generated for grade match reliability assessment

### Key Design Decisions
- **Real-Time Processing**: Grade matching occurs automatically with each chemistry update
- **Statistical Confidence**: nSigma calculations provide uncertainty-based grade boundaries
- **Multiple Library Support**: Factory, user, and residuals libraries for flexibility
- **Mode Integration**: Grade matching adapts to current test mode (PMI/reCycling)
- **Visual Feedback**: Color-coded displays for immediate match status recognition
- **Comprehensive Logging**: All grade match results stored for audit and traceability

### Success Criteria
- Grade matching results update automatically with live chemistry
- Match numbers calculated accurately with statistical confidence
- Confidence indicators display alongside grade match results in real-time
- Measurement uncertainty values (± readings) shown with elemental results
- Confidence-based color coding provides intuitive grade match quality visualization
- Statistical boundary visualizations clearly indicate confidence ranges
- Multiple grade libraries supported with appropriate filtering
- Visual grade match indicators provide clear pass/fail status with confidence levels
- Residual element analysis integrated with grade matching
- Application-specific confidence thresholds properly applied (PMI vs scrap sorting)
- All grade match results with confidence data properly stored for traceability

---

## Use Case 2: Alloy Mismatch Detection

### Overview
Intelligent mismatch detection system that identifies samples failing to meet any grade library specifications. System analyzes grade match failures, detects root causes (coating, contamination, insufficient test time), provides diagnostic information, and suggests corrective actions to guide users toward proper sample analysis.

### Sequence Flow

```
Actor: System (Triggered by Grade Match Results)
UI Layer: User Interface
MW API: Middleware API
DB: Database
```

**Step-by-Step API Flow:**

### Mismatch Detection Process (Triggered from Grade Match)

1. **Grade Match Result Analysis**
   ```
   MW API: Receive grade match results from Grade Match use case
   MW API: Check if any grades meet match cutoff threshold criteria
   MW API: Determine overall match status (match/no_match)
   ```

2. **Mismatch Classification**
   ```
   MW API: If no acceptable matches found, initiate mismatch detection
   MW API -> DB: getGradeLibrarySpecs(current_mode, base_material_type)
   DB -> MW API: Return complete grade specifications for analysis
   MW API: Compare chemistry against ALL grade library specifications
   MW API: Validate match cutoff threshold compliance across all grades
   ```

3. **Root Cause Analysis**
   ```
   MW API: Analyze spectral patterns for coating detection indicators
   MW API: Process residual element concentrations against base material tolerances
   MW API: Evaluate measurement time vs. detection limit requirements
   MW API: Check for contamination patterns in elemental ratios
   MW API: Assess statistical confidence levels for reliable detection
   ```

4. **Residual Element Processing**
   ```
   MW API: Process residual (tramp) elements by base material type
   MW API: Compare residual concentrations against tolerance thresholds
   MW API: Identify elements exceeding acceptable limits
   MW API: Flag unexpected elements not typical for base material
   ```

5. **Coating Detection Analysis**
   ```
   MW API: Analyze spectral signatures for surface coating presence
   MW API: Detect coating indicators (Zn, Cr, Ni surface signals vs bulk)
   MW API: Evaluate surface-to-bulk elemental ratios
   MW API: Identify coating thickness and composition effects
   ```

6. **Diagnostic Information Generation**
   ```
   MW API: Determine primary mismatch reasons from analysis
   MW API: Generate severity indicators for mismatch confidence
   MW API: Create diagnostic recommendations based on root cause
   MW API: Identify partial matches and alternative grade suggestions
   MW API: Generate suggested corrective actions
   ```

7. **Mismatch Results Compilation**
   ```
   MW API: Compile comprehensive mismatch report
   MW API: Include primary and secondary mismatch reasons
   MW API: Add residual element alerts and concentration warnings
   MW API: Include coating detection results and surface condition alerts
   MW API: Prepare suggested actions (extend test time, clean sample, service)
   MW API: Generate alternative grade suggestions for partial matches
   ```

8. **Live Mismatch Display**
   ```
   MW API -> UI: displayMismatchResults(mismatch_data, diagnostics, suggestions) (async)
   UI: Display "No Match" results prominently in Live View
   UI: Show mismatch reasons and diagnostic information
   UI: Present residual (tramp) element alerts and concentration warnings
   UI: Display coating detection indicators and surface condition alerts
   UI: Show suggested actions for mismatch resolution
   UI: Provide mismatch severity indicators and detection confidence
   UI: Display alternative grade suggestions when partial matches exist
   ```

9. **Mismatch Event Logging**
   ```
   MW API -> DB: logMismatchEvent(session_id, mismatch_reasons, diagnostics, timestamp)
   ```

### API Sequence Diagram

```
System   MW API            DB                UI
 |        |                 |                 |
 |        | [grade match results] |          |  1. Receive match results (no acceptable matches)
 |        |---------------->|                |  2. getGradeLibrarySpecs()
 |        |<----------------|                |  3. Return complete grade specifications
 |        | [mismatch classification] |      |  4. Classify mismatch type & severity
 |        | [root cause analysis] |         |  5. Analyze coating, residuals, contamination
 |        | [residual processing] |         |  6. Process tramp elements vs tolerances
 |        | [coating detection] |           |  7. Detect surface coating presence
 |        | [generate diagnostics] |        |  8. Create diagnostic recommendations
 |        |--------------------------------->|  9. displayMismatchResults()
 |        |                 |                | [show] 10. Display "No Match" with diagnostics
 |        |---------------->|                | 11. logMismatchEvent()
 |        |                 |                |
```

### Mismatch Detection Categories

**Primary Mismatch Types:**
- **No Library Match**: Sample chemistry doesn't match any grade in selected libraries
- **Threshold Violations**: All matches exceed maximum acceptable match number cutoff
- **Confidence Insufficient**: Matches exist but confidence levels too low for reliable identification

**Root Cause Categories:**
- **Coating Presence**: Surface coatings (galvanized, plated, painted) affecting analysis
- **Contamination**: Foreign material contamination or surface debris
- **Residual Elements**: Tramp elements exceeding base material specifications
- **Insufficient Test Time**: Short measurement time resulting in poor statistics
- **Sample Preparation**: Poor surface preparation or sample positioning issues
- **Base Material Unknown**: Sample not compatible with selected grade libraries

### Diagnostic Recommendations

**Coating Detection Actions:**
- **Clean Sample**: Remove surface coatings, paint, or debris
- **Grind Surface**: Remove surface layer to expose bulk material
- **Extended Test**: Longer measurement time to penetrate coating

**Contamination Actions:**
- **Sample Cleaning**: Remove foreign material and debris
- **Reposition Sample**: Ensure proper sample-to-detector alignment
- **Check Sample Prep**: Verify sample preparation procedures

**Statistical Actions:**
- **Extend Test Time**: Increase measurement duration for better statistics
- **Repeat Measurement**: Multiple measurements for statistical validation
- **Check Calibration**: Verify instrument calibration status

**Library Actions:**
- **Change Library**: Select different grade library appropriate for material
- **Custom Grade**: Define custom grade specification if known material
- **Base Material Check**: Verify base material type selection

### Alternative Grade Suggestions
- **Partial Matches**: Grades meeting some but not all element specifications
- **Close Matches**: Grades with match numbers slightly above cutoff threshold
- **Similar Chemistry**: Grades with comparable elemental compositions
- **Library Recommendations**: Suggest alternative grade libraries to check

### Key Design Decisions
- **Real-Time Analysis**: Mismatch detection occurs immediately after grade matching
- **Comprehensive Diagnostics**: Multiple root cause analysis algorithms
- **User Guidance**: Clear diagnostic information and suggested corrective actions
- **Spectral Analysis**: Advanced coating and contamination detection
- **Confidence-Based**: Mismatch detection considers statistical confidence levels
- **Actionable Results**: Specific recommendations for resolving mismatch issues

### Success Criteria
- Mismatch detection activates when no grades meet acceptance criteria
- Root cause analysis accurately identifies primary mismatch reasons
- Coating detection reliably identifies surface contamination issues
- Residual element processing flags tramp elements exceeding tolerances
- Diagnostic recommendations provide actionable guidance for users
- Alternative grade suggestions help identify partial matches
- Mismatch events properly logged for quality control analysis
- UI clearly displays mismatch status with comprehensive diagnostic information

---

## Use Case 3: Show GradeMatch Pass/Fail

### Overview
Automated Pass/Fail determination system for quality control applications. System compares measured chemistry against a selected reference grade using statistical confidence boundaries and match number thresholds to provide clear Pass/Fail decisions for high-throughput sorting and compliance verification.

### Sequence Flow

```
Actor: System (Triggered by Grade Match Results)
UI Layer: User Interface
MW API: Middleware API
DB: Database
```

**Step-by-Step API Flow:**

### Pass/Fail Configuration Setup

1. **Pass/Fail Settings Access**
   ```
   User -> UI: Navigate to Pass/Fail configuration screen
   UI -> MW API: GET /api/pass-fail/configuration (async)
   MW API -> DB: getPassFailSettings(current_method, test_mode)
   DB -> MW API: Return current Pass/Fail configuration and thresholds
   MW API -> UI: Return pass_fail_config (reference_grade, thresholds, nSigma_settings)
   UI: Display Pass/Fail configuration screen with current settings
   ```

2. **Reference Grade Selection**
   ```
   User -> UI: Select reference grade from base element categories
   UI -> MW API: POST /api/pass-fail/set-reference-grade (selected_grade) (async)
   MW API: Validate reference grade selection against current test mode
   MW API: Load reference grade specifications and tolerance ranges
   MW API -> DB: savePassFailConfiguration(reference_grade, method_id)
   MW API -> UI: Return reference_grade_status
   UI: Display selected reference grade with specifications
   ```

3. **Pass/Fail Threshold Configuration**
   ```
   User -> UI: Configure match number cutoffs, nSigma boundaries, tolerance ranges
   UI -> MW API: POST /api/pass-fail/configure-thresholds (threshold_settings) (async)
   MW API: Validate threshold settings for statistical validity
   MW API: Store Pass/Fail settings per test method and application mode
   MW API -> DB: savePassFailThresholds(method_id, thresholds, nSigma_settings)
   MW API -> UI: Return threshold_config_status
   ```

### Live Pass/Fail Determination Process

4. **Chemistry Data Reception** (From Live Chemistry/Grade Match Use Cases)
   ```
   MW API: Receive new chemistry data and grade match results
   MW API: Check if Pass/Fail mode is enabled and reference grade is selected
   MW API: Extract measured concentrations and uncertainties
   ```

5. **Reference Grade Comparison**
   ```
   MW API -> DB: getReferenceGradeSpecs(selected_reference_grade)
   DB -> MW API: Return reference grade specifications (elements, nominal, tolerances)
   MW API: Compare measured chemistry against reference grade specifications
   MW API: Calculate match number for reference grade comparison
   ```

6. **Pass/Fail Calculation with Statistical Confidence**
   ```
   MW API: Apply nSigma boundaries to Pass/Fail criteria for statistical confidence
   MW API: Calculate expanded tolerance ranges (grade_tolerance ± nSigma × uncertainty)
   MW API: Process Pass/Fail logic against selected reference grade specifications
   MW API: Evaluate each element against Pass/Fail criteria
   MW API: Apply match number threshold to overall Pass/Fail determination
   ```

7. **Pass/Fail State Determination**
   ```
   MW API: Determine overall Pass/Fail status based on all criteria
   MW API: Handle Pass/Fail state transitions and result validation
   MW API: Calculate confidence level for Pass/Fail determination
   MW API: Generate Pass/Fail reasoning (which elements passed/failed)
   ```

8. **Pass/Fail Results Generation**
   ```
   MW API: Compile comprehensive Pass/Fail results
   MW API: Include overall Pass/Fail status, confidence level, match number
   MW API: Add element-by-element Pass/Fail breakdown
   MW API: Include statistical boundaries and threshold comparisons
   MW API: Coordinate Pass/Fail results with grade comparison features
   ```

9. **Live Pass/Fail Display**
   ```
   MW API -> UI: updatePassFailResults(pass_fail_status, details, confidence) (async)
   UI: Display prominent Pass/Fail indicators in Live View with color coding (green/red)
   UI: Show Pass/Fail status alongside grade comparison results
   UI: Present Pass/Fail criteria with match number cutoffs and tolerance ranges
   UI: Display element-by-element Pass/Fail breakdown
   UI: Show confidence level and statistical boundaries
   ```

10. **Pass/Fail Results Storage**
    ```
    MW API -> DB: savePassFailResults(session_id, pass_fail_status, confidence, reference_grade)
    MW API -> DB: linkPassFailToTestConditions(session_id, calibration_status, test_conditions)
    ```

### API Sequence Diagram

```
System   MW API            DB                UI
 |        |                 |                 |
 |        | [chemistry & grade match received] |  1. Receive chemistry/grade match data
 |        |---------------->|                |  2. getReferenceGradeSpecs()
 |        |<----------------|                |  3. Return reference grade specifications
 |        | [compare chemistry] |            |  4. Compare measured vs reference grade
 |        | [calc pass/fail] |               |  5. Apply nSigma boundaries & thresholds
 |        | [state determination] |          |  6. Determine overall Pass/Fail status
 |        | [generate results] |             |  7. Compile Pass/Fail results with confidence
 |        |--------------------------------->|  8. updatePassFailResults()
 |        |                 |                | [display] 9. Show Pass/Fail with color coding
 |        |---------------->|                | 10. savePassFailResults()
 |        |                 |                |
```

### Pass/Fail Determination Logic

**Pass Criteria (ALL must be met):**
- Match number against reference grade ≤ configured threshold
- All critical elements within nSigma expanded tolerance ranges
- Statistical confidence level meets minimum requirements
- No major element violations exceed allowable limits

**Fail Criteria (ANY triggers failure):**
- Match number against reference grade > configured threshold
- Any critical element outside nSigma expanded tolerance ranges
- Statistical confidence insufficient for reliable determination
- Major element concentrations violate specification limits

**Inconclusive Criteria:**
- Insufficient measurement time for statistical reliability
- Measurement uncertainty too large for confident determination
- Reference grade specifications incomplete for comparison

### Pass/Fail Configuration Options

**Reference Grade Selection:**
- **Base Element Categories**: Steel, Aluminum, Copper, Titanium, etc.
- **Specific Grades**: User-selectable from grade libraries
- **Custom Reference**: User-defined reference specifications
- **Multiple References**: Different reference grades per test method

**Threshold Settings:**
- **Match Number Cutoff**: Maximum acceptable match number for Pass
- **nSigma Boundaries**: Statistical confidence multiplier (typically 1-3 sigma)
- **Element-Specific**: Individual thresholds per element type
- **Confidence Minimum**: Required statistical confidence for valid determination

### Application-Specific Settings
- **PMI Mode**: Strict thresholds for material verification
- **reCycling Mode**: Relaxed thresholds for scrap sorting
- **QC Mode**: Tightest thresholds for quality control
- **Custom Applications**: User-configurable threshold sets

### Key Design Decisions
- **Statistical Confidence**: nSigma boundaries provide uncertainty-based Pass/Fail criteria
- **Reference Grade Based**: Clear comparison against known standard
- **Real-Time Processing**: Pass/Fail determination occurs with each chemistry update
- **Color-Coded Display**: Immediate visual feedback (green/red) for operators
- **Configurable Thresholds**: Adaptable to different quality requirements
- **Comprehensive Logging**: Complete audit trail for quality control records

### Success Criteria
- Pass/Fail determination updates automatically with live chemistry results
- Reference grade comparison accurately applies statistical confidence boundaries
- Color-coded Pass/Fail indicators provide immediate visual feedback
- Element-by-element Pass/Fail breakdown helps identify specific issues
- Pass/Fail thresholds properly configured per test method and application
- Statistical confidence levels ensure reliable Pass/Fail decisions
- All Pass/Fail results linked to test conditions and calibration status
- Comprehensive audit trail maintained for quality control compliance

---

## Use Case 4: Scrap Type Classification

### Overview
Specialized scrap metal classification system optimized for recycling operations. System leverages the existing Grade Match engine with scrap-specific grade libraries and classification hierarchies to provide fast, accurate scrap type identification with confidence levels and value metrics for efficient material sorting.

### Sequence Flow

```
Actor: System (Continuous Process in reCycling Mode)
UI Layer: User Interface
MW API: Middleware API
DB: Database
```

**Step-by-Step API Flow:**

### Scrap Classification Configuration

1. **Scrap Library Configuration**
   ```
   User -> UI: Navigate to scrap type configuration screen
   UI -> MW API: GET /api/scrap-classification/configuration (async)
   MW API -> DB: getScrapConfiguration(recycling_facility_id, current_mode)
   DB -> MW API: Return scrap library settings and classification parameters
   MW API -> UI: Return scrap_config (libraries, thresholds, facility_settings)
   UI: Display scrap type configuration screen with library selection
   ```

2. **Scrap Library Selection**
   ```
   User -> UI: Select scrap-specific grade libraries and classification hierarchies
   UI -> MW API: POST /api/scrap-classification/set-libraries (selected_libraries) (async)
   MW API: Validate library selection against reCycling mode requirements
   MW API: Load scrap type grade libraries and classification rules
   MW API -> DB: saveScrapConfiguration(facility_id, libraries, thresholds)
   MW API -> UI: Return library_selection_status
   ```

3. **Classification Threshold Configuration**
   ```
   User -> UI: Configure classification confidence thresholds and value metrics
   UI -> MW API: POST /api/scrap-classification/configure-thresholds (threshold_settings) (async)
   MW API: Store scrap type configuration parameters per recycling facility
   MW API: Coordinate with recycling mode settings for adaptive testing
   MW API -> DB: saveClassificationThresholds(facility_id, thresholds)
   ```

### Live Scrap Classification Process

4. **Chemistry Data Reception** (From Live Chemistry Use Case)
   ```
   MW API: Receive new chemistry data from live analysis (reCycling mode active)
   MW API: Check if scrap classification is enabled
   MW API: Extract elemental concentrations and uncertainties for scrap analysis
   ```

5. **Scrap-Specific Grade Library Loading**
   ```
   MW API -> DB: loadScrapGradeLibraries(facility_id, classification_hierarchy)
   DB: Query scrap-specific grade libraries (Steel, Aluminum, Copper, etc.)
   DB: Filter by base material type and scrap categories
   DB -> MW API: Return scrap_grade_specifications (scrap_types, value_grades, tolerances)
   ```

6. **Scrap Classification using Grade Match Engine**
   ```
   MW API: Leverage existing Grade Match engine for scrap type determination
   MW API: Compare measured chemistry against scrap grade specifications
   MW API: Calculate match numbers for each scrap type category
   MW API: Apply scrap-specific classification rules and tolerances
   MW API: Generate classification_results (scrap_type, match_score, confidence)
   ```

7. **Scrap Type Confidence Scoring**
   ```
   MW API: Handle scrap type confidence scoring and classification reliability
   MW API: Apply recycling-optimized confidence thresholds
   MW API: Calculate value metrics based on scrap type and market data
   MW API: Evaluate classification reliability for high-throughput sorting
   MW API: Generate confidence_assessment (high/medium/low confidence)
   ```

8. **Classification Hierarchy Processing**
   ```
   MW API: Process results through scrap classification hierarchies
   MW API: Determine primary scrap category (Ferrous/Non-Ferrous)
   MW API: Identify specific scrap type (304 SS, 6061 Al, Brass, etc.)
   MW API: Apply contamination detection for mixed scrap
   MW API: Generate alternative classifications for borderline cases
   ```

9. **Scrap Value Assessment**
   ```
   MW API: Calculate scrap value metrics based on classification
   MW API: Apply facility-specific pricing and market factors
   MW API: Generate sorting recommendations and value indicators
   MW API: Include contamination warnings that affect value
   ```

10. **Live Scrap Classification Display**
    ```
    MW API -> UI: updateScrapClassification(scrap_data, confidence, value_metrics) (async)
    UI: Display scrap type classification results in Live View
    UI: Show scrap type categories with confidence levels and match scores
    UI: Present classification hierarchy (category -> type -> grade)
    UI: Display value metrics and sorting recommendations
    UI: Show contamination warnings and purity indicators
    ```

11. **Scrap Classification Storage**
    ```
    MW API -> DB: saveScrapClassification(session_id, scrap_type, confidence, value_metrics)
    MW API -> DB: updateScrapStatistics(facility_id, daily_totals, value_summary)
    ```

### API Sequence Diagram

```
System   MW API            DB                UI
 |        |                 |                 |
 |        | [chemistry received - reCycling mode] | 1. Receive chemistry data
 |        |---------------->|                |  2. loadScrapGradeLibraries()
 |        |<----------------|                |  3. Return scrap grade specifications
 |        | [use Grade Match engine] |       |  4. Leverage existing grade matching
 |        | [scrap classification] |         |  5. Apply scrap-specific rules & hierarchies
 |        | [confidence scoring] |           |  6. Calculate classification confidence
 |        | [value assessment] |             |  7. Determine scrap value metrics
 |        |--------------------------------->|  8. updateScrapClassification()
 |        |                 |                | [display] 9. Show scrap type & value
 |        |---------------->|                | 10. saveScrapClassification()
 |        |                 |                |
```

### Scrap Classification Hierarchies

**Primary Categories:**
- **Ferrous Scrap**: Steel types, stainless steels, cast iron
- **Non-Ferrous Scrap**: Aluminum, copper, brass, zinc, lead
- **Precious Metals**: Gold, silver, platinum group metals
- **Specialty Alloys**: Titanium, nickel-based alloys, exotic metals

**Classification Levels:**
- **Level 1**: Base Material (Fe, Al, Cu, etc.)
- **Level 2**: Alloy Family (Carbon Steel, Stainless, 6xxx Al, etc.)
- **Level 3**: Specific Grade (304 SS, 6061 Al, C101 Cu, etc.)
- **Level 4**: Quality/Purity (Clean, Contaminated, Mixed)

### Scrap-Specific Features

**High-Throughput Optimization:**
- **Fast Classification**: Optimized for rapid material sorting
- **Confidence Thresholds**: Relaxed confidence levels for speed
- **Value-Based Sorting**: Economic value considerations integrated
- **Contamination Detection**: Mixed scrap and contamination alerts

**Recycling Facility Integration:**
- **Facility-Specific Libraries**: Custom scrap types per facility
- **Market Value Integration**: Real-time pricing and value metrics
- **Daily Statistics**: Running totals and value summaries
- **Sorting Recommendations**: Bin assignments and handling instructions

### Grade Match Engine Leverage
- **Existing Infrastructure**: Reuses Grade Match algorithms and confidence calculations
- **Scrap-Optimized Libraries**: Specialized grade libraries for recycling applications
- **Modified Thresholds**: Relaxed match criteria for faster throughput
- **Enhanced Categories**: Extended classification hierarchies for scrap types

### Key Design Decisions
- **Existing Engine Leverage**: Reuses proven Grade Match infrastructure for consistency
- **Scrap-Optimized**: Modified algorithms and thresholds for recycling applications
- **Value Integration**: Economic considerations built into classification process
- **High-Throughput Design**: Optimized for fast material sorting operations
- **Facility Customization**: Configurable per recycling facility requirements
- **Statistical Confidence**: Adapted confidence levels for recycling speed requirements

### Success Criteria
- Scrap classification leverages existing Grade Match engine effectively
- Classification results display with confidence levels and value metrics
- Scrap-specific libraries and hierarchies provide accurate categorization
- High-throughput performance suitable for recycling operations
- Value assessment helps optimize sorting and revenue
- Facility-specific configuration supports diverse recycling operations
- Integration with reCycling mode settings and adaptive testing
- Comprehensive classification statistics for facility management

---

## Use Case 5: Adaptive Time Testing

### Overview
Intelligent adaptive testing system that dynamically adjusts measurement parameters and test duration based on real-time spectrum analysis and light element detection requirements. System uses calcEngine feedback to optimize beam parameters and extend test time for improved light element detection and measurement reliability.

### Sequence Flow

```
Actor: System (Continuous Process During Active Scan)
UI Layer: User Interface
MW API: Middleware API
Calc Engine: Chemistry Calculation Engine
HW API: Hardware API
DB: Database
```

**Step-by-Step API Flow:**

### Adaptive Testing Configuration

1. **Adaptive Mode Configuration**
   ```
   User -> UI: Navigate to adaptive testing configuration screen
   UI -> MW API: GET /api/adaptive-testing/configuration (async)
   MW API -> DB: getAdaptiveSettings(current_method, test_mode)
   DB -> MW API: Return adaptive configuration (enable/disable, thresholds, max_extensions)
   MW API -> UI: Return adaptive_config
   UI: Display adaptive mode configuration settings
   ```

2. **Adaptive Threshold Configuration**
   ```
   User -> UI: Configure detection thresholds, max extensions, beam parameter limits
   UI -> MW API: POST /api/adaptive-testing/configure (adaptive_settings) (async)
   MW API: Validate adaptive decisions against hardware and safety constraints
   MW API: Store adaptive testing configuration and threshold management
   MW API -> DB: saveAdaptiveConfiguration(method_id, thresholds, constraints)
   MW API -> UI: Return configuration_status
   ```

### Live Adaptive Testing Process

3. **Scan Initiation with Adaptive Monitoring**
   ```
   MW API: Begin scan with adaptive testing enabled
   MW API: Initialize adaptive testing state machine and decision logic
   MW API: Set initial beam parameters and test duration
   MW API: Start adaptive monitoring process
   ```

4. **Real-Time Spectrum Analysis** (Continuous during scan)
   ```
   HW API -> MW API: Provide live spectrum data stream
   MW API -> Calc Engine: analyzeSpectrumForAdaptation(current_spectrum, elapsed_time) (async)
   Calc Engine: Process light element detection probability assessments
   Calc Engine: Analyze spectrum quality and statistical confidence
   Calc Engine: Evaluate need for parameter adjustments or time extensions
   Calc Engine -> MW API: Return adaptation_recommendations (light_element_probability, suggested_actions)
   ```

5. **Adaptive Decision Processing**
   ```
   MW API: Process calcEngine spectrum analysis results
   MW API: Evaluate light element detection probability against thresholds
   MW API: Assess current measurement confidence and quality metrics
   MW API: Apply adaptive testing decision logic based on calcEngine recommendations
   MW API: Determine if beam parameter changes or time extensions are needed
   ```

6. **Beam Parameter Adaptation** (if recommended by calcEngine)
   ```
   MW API: Generate adaptive beam parameter commands based on calcEngine feedback
   MW API -> HW API: adaptBeamParameters(new_voltage, new_current, safety_checks) (sync)
   HW API: Validate safety interlocks during adaptive parameter changes
   HW API: Execute tube voltage and current adjustments during testing
   HW API: Monitor beam stability during parameter transitions
   HW API -> MW API: Return parameter_change_status (SUCCESS/FAILED, stability_metrics)
   ```

7. **Test Time Extension Management**
   ```
   MW API: Evaluate test time extensions based on calcEngine recommendations
   MW API: Check against maximum extension limits and safety constraints
   MW API: Calculate dynamic time estimates for improved light element detection
   MW API: Extend measurement duration if within configured limits
   MW API: Update adaptive testing progress tracking
   ```

8. **Live Adaptive Feedback Display**
   ```
   MW API -> UI: updateAdaptiveProgress(time_estimates, detection_probability, beam_changes) (async)
   UI: Display adaptive testing progress with dynamic time estimates in Live View
   UI: Present real-time feedback on light element detection probability
   UI: Display beam parameter changes and adaptation status
   UI: Show adaptive testing progress indicators and remaining time estimates
   ```

9. **Adaptive Completion Assessment**
   ```
   MW API: Continuously evaluate measurement completion criteria
   MW API: Check if light element detection targets have been met
   MW API: Validate final measurement quality against adaptive thresholds
   MW API: Determine when adaptive testing objectives are satisfied
   ```

10. **Adaptive Results Storage**
    ```
    MW API -> DB: saveAdaptiveResults(session_id, adaptations_made, final_parameters, detection_success)
    MW API -> DB: logAdaptiveDecisions(decision_log, calcEngine_recommendations, outcomes)
    ```

### API Sequence Diagram

```
System   MW API    Calc Engine    HW API    UI       DB
 |        |         |              |         |        |
 |        | [scan active - adaptive enabled] |        |  1. Adaptive scan initiated
 |        |-------->|              |         |        |  2. analyzeSpectrumForAdaptation()
 |        |         | [analyze]    |         |        |  3. Process light element detection
 |        |<--------|              |         |        |  4. Return adaptation_recommendations
 |        | [decision logic]       |         |        |  5. Process calcEngine feedback
 |        |-------------------->   |         |        |  6. adaptBeamParameters() (if needed)
 |        |<--------------------|   |         |        |  7. Return parameter_change_status
 |        | [time extension]       |         |        |  8. Manage test time extensions
 |        |--------------------------------->|        |  9. updateAdaptiveProgress()
 |        |                      |         | [show] | 10. Display adaptive feedback
 |        |--------------------------------------------->| 11. saveAdaptiveResults()
 |        |                      |         |        |
```

### Adaptive Testing State Machine

**States:**
- **Adaptive Monitoring**: Continuous spectrum analysis and evaluation
- **Parameter Adaptation**: Adjusting beam parameters based on calcEngine feedback
- **Time Extension**: Extending measurement duration for better detection
- **Stability Validation**: Ensuring beam stability after parameter changes
- **Completion Assessment**: Evaluating if adaptive objectives are met

**Decision Triggers:**
- **Light Element Probability**: Below threshold triggers parameter/time adaptations
- **Statistical Confidence**: Insufficient confidence extends measurement time
- **calcEngine Recommendations**: Direct feedback drives adaptation decisions
- **Safety Constraints**: Hardware limits prevent unsafe adaptations

### Adaptive Parameters

**Beam Parameter Adaptations:**
- **Tube Voltage**: Optimized for light element excitation
- **Tube Current**: Adjusted for improved count rates
- **Filter Selection**: Modified for better light element detection
- **Detector Settings**: Optimized for spectral quality

**Time Extension Parameters:**
- **Maximum Extensions**: Configurable limits per test method
- **Extension Increments**: Time added per adaptation cycle
- **Detection Thresholds**: Light element probability targets
- **Quality Metrics**: Statistical confidence requirements

### CalcEngine Integration
- **Real-Time Analysis**: Continuous spectrum evaluation during measurement
- **Light Element Focus**: Specialized analysis for low-Z element detection
- **Adaptive Recommendations**: Specific suggestions for parameter/time adjustments
- **Probability Assessment**: Statistical evaluation of detection likelihood

### Safety and Constraints
- **Hardware Limits**: Beam parameter changes within safe operating ranges
- **Safety Interlocks**: Radiation safety maintained during adaptations
- **Time Limits**: Maximum test duration constraints
- **Stability Requirements**: Beam stability validation after parameter changes

### Key Design Decisions
- **CalcEngine Driven**: Adaptation decisions based on real-time spectrum analysis
- **Real-Time Feedback**: Continuous UI updates on adaptive progress
- **Safety First**: All adaptations validated against safety constraints
- **Light Element Focus**: Optimized for challenging light element detection
- **State Machine Control**: Systematic management of adaptive testing phases
- **Configurable Limits**: User-definable thresholds and maximum extensions

### Success Criteria
- Adaptive testing responds to calcEngine spectrum analysis recommendations
- Beam parameter adaptations improve light element detection probability
- Test time extensions optimize measurement quality within configured limits
- Real-time UI feedback shows adaptive progress and parameter changes
- Safety constraints prevent unsafe adaptations during testing
- Adaptive decisions improve overall measurement reliability and confidence
- Integration with existing scan workflow maintains system consistency
- Comprehensive logging enables adaptive performance analysis and optimization

---