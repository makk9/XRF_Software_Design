# XRF Analyzer System - System Use Cases

## Use Case 1: Login

### Overview
User authentication system that validates user credentials when powering on the device and provides access control to the XRF analyzer system. System verifies username and password against stored user database and transitions to main application interface upon successful authentication.

### Sequence Flow

```
Actor: User
UI Layer: User Interface
MW API: Middleware API
DB: Database
```

**Step-by-Step API Flow:**

### Device Power-On and Login Display

1. **System Startup**
   ```
   System: Device powered on
   UI: Initialize login screen interface
   UI: Display login form (username, password fields)
   UI: Show system branding and version information
   ```

2. **User Credential Entry**
   ```
   User -> UI: Enter username and password
   User -> UI: Press login/submit button
   ```

3. **Login Request Submission**
   ```
   UI -> MW API: POST /api/auth/login (username, password) (async)
   ```

4. **Credential Validation**
   ```
   MW API -> DB: validateUser(username, password_hash)
   DB: Query user database for matching credentials
   DB: Verify password hash against stored value
   DB: Check user account status (active, locked, expired)
   DB -> MW API: Return validation_result (success/failure, user_details, permissions)
   ```

5. **Authentication Processing**
   ```
   MW API: Process authentication result
   MW API: Generate session token if authentication successful
   MW API: Log authentication attempt (success/failure)
   MW API: Prepare validation response with user permissions
   ```

6. **Login Response**
   ```
   MW API -> UI: Return login_validation (success/failure, session_token, user_info)
   ```

7. **UI State Transition**
   ```
   If authentication successful:
     UI: Store session token for subsequent API calls
     UI: Load user preferences and settings
     UI: Transition from login screen to homescreen/main interface
   
   If authentication failed:
     UI: Display error message (invalid credentials, account locked, etc.)
     UI: Clear password field
     UI: Remain on login screen for retry
   ```

8. **Session Initialization** (On successful login)
   ```
   MW API -> DB: createUserSession(user_id, session_token, login_timestamp)
   MW API: Initialize user-specific system settings
   MW API: Load user permissions and access levels
   ```

### API Sequence Diagram

```
User    UI       MW API            DB
 |       |         |                |
 |------>|         |                |  1. Enter credentials & press login
 |       |-------->|                |  2. POST /api/auth/login
 |       |         |--------------->|  3. validateUser()
 |       |         |<---------------|  4. Return validation_result
 |       |         | [process auth] |  5. Generate session & log attempt
 |       |<--------|                |  6. Return login_validation
 |       | [transition to home]     |  7. Switch to homescreen (if success)
 |       |         |--------------->|  8. createUserSession() (if success)
 |       |         |                |
```

### Authentication Security Features

**Password Security:**
- **Hash Storage**: Passwords stored as cryptographic hashes, never plaintext
- **Salt Protection**: Unique salt per password prevents rainbow table attacks
- **Iteration Count**: Multiple hash iterations increase computational cost

**Account Protection:**
- **Login Attempts**: Failed login attempt counting and temporary lockout
- **Account Status**: Support for active, locked, expired, and disabled accounts
- **Session Management**: Secure session tokens with expiration times

**Audit Trail:**
- **Login Logging**: All authentication attempts logged with timestamps
- **User Activity**: Session creation and user actions tracked
- **Security Events**: Failed login attempts and security violations recorded

### User Management Features

**User Types:**
- **Administrator**: Full system access and user management
- **Operator**: Standard measurement and analysis functions
- **Technician**: Calibration and maintenance functions
- **Viewer**: Read-only access to results and data

**User Preferences:**
- **Interface Settings**: UI layout and display preferences
- **Default Configurations**: Preferred test modes and settings
- **Language Selection**: Localization preferences
- **Notification Settings**: Alert and warning preferences

### Error Handling

**Authentication Failures:**
- **Invalid Credentials**: Username/password combination not found
- **Account Locked**: Too many failed login attempts
- **Account Expired**: User account past expiration date
- **Account Disabled**: Administrator disabled account
- **System Error**: Database connection or system issues

**User Feedback:**
- **Clear Error Messages**: Specific reason for login failure
- **Security Considerations**: Limited information to prevent account enumeration
- **Retry Guidance**: Instructions for resolving login issues
- **Contact Information**: Support contact for account issues

### Key Design Decisions
- **Security First**: Passwords hashed and salted, secure session management
- **User Experience**: Clear login interface with helpful error messages
- **Audit Compliance**: Comprehensive logging for regulatory requirements
- **Role-Based Access**: User permissions control system functionality
- **Session Management**: Secure tokens with automatic expiration
- **Database Integration**: Centralized user management and authentication

### Success Criteria
- User successfully authenticates with valid credentials
- Invalid credentials properly rejected with appropriate error messages
- Session token generated and stored for subsequent API authentication
- UI transitions from login screen to homescreen upon successful authentication
- Failed login attempts logged for security monitoring
- User permissions and preferences loaded for personalized experience
- Account lockout protection prevents brute force attacks
- Comprehensive audit trail maintained for compliance requirements

---

## Use Case 2: Logout

### Overview
User session termination system that safely logs out the current user, clears session data, and returns to the login screen. System ensures proper cleanup of user session, saves any pending data, and provides security through session invalidation.

### Sequence Flow

```
Actor: User
UI Layer: User Interface
MW API: Middleware API
DB: Database
```

**Step-by-Step API Flow:**

### Logout Initiation

1. **Logout Button Access**
   ```
   User -> UI: Navigate to menu bar/settings
   UI: Display logout button in menu options
   ```

2. **Logout Request**
   ```
   User -> UI: Press logout button
   ```

3. **Logout Confirmation** (Optional safety check)
   ```
   UI: Display confirmation dialog "Are you sure you want to logout?"
   User -> UI: Confirm logout action
   ```

4. **Pre-Logout Safety Checks**
   ```
   UI: Check for active scan operations
   UI: Check for unsaved data or pending operations
   
   If active scan in progress:
     UI: Display warning "Scan in progress. Logout will stop current measurement."
     User -> UI: Confirm or cancel logout
   
   If unsaved data exists:
     UI: Display option to save pending data before logout
   ```

5. **Logout Request Submission**
   ```
   UI -> MW API: POST /api/auth/logout (session_token) (async)
   ```

6. **Session Cleanup and Invalidation**
   ```
   MW API: Validate current session token
   MW API -> DB: invalidateUserSession(session_token, logout_timestamp)
   DB: Mark session as terminated in database
   DB: Record logout timestamp and reason
   DB -> MW API: Return session_invalidation_status
   ```

7. **System State Cleanup**
   ```
   MW API: Clear user-specific cached data
   MW API: Save any pending user preferences or settings
   MW API: Stop any user-specific background processes
   MW API: Reset system to default/login state
   MW API: Log logout event for audit trail
   ```

8. **Logout Response**
   ```
   MW API -> UI: Return logout_confirmation (success/failure, cleanup_status)
   ```

9. **UI State Transition**
   ```
   UI: Clear stored session token
   UI: Clear user-specific UI state and preferences
   UI: Reset interface to default login state
   UI: Transition from current screen to login screen
   UI: Display login form for new authentication
   ```

10. **Security Cleanup**
    ```
    UI: Clear sensitive data from memory
    UI: Reset any cached user information
    UI: Ensure no user data remains accessible
    ```

### API Sequence Diagram

```
User    UI       MW API            DB
 |       |         |                |
 |------>|         |                |  1. Press logout button
 |       | [confirmation dialog]    |  2. Confirm logout (optional)
 |       | [safety checks]          |  3. Check active scans/unsaved data
 |------>|         |                |  4. Confirm after safety check
 |       |-------->|                |  5. POST /api/auth/logout
 |       |         |--------------->|  6. invalidateUserSession()
 |       |         |<---------------|  7. Return session_invalidation_status
 |       |         | [cleanup]      |  8. Clear cached data & log event
 |       |<--------|                |  9. Return logout_confirmation
 |       | [clear session & UI]     | 10. Transition to login screen
 |       |         |                |
```

### Safety and Data Protection

**Active Scan Protection:**
- **Scan Detection**: Check for active X-ray emission or measurement in progress
- **Warning Display**: Alert user that logout will terminate active measurements
- **Graceful Termination**: Safely stop scan operations before logout
- **Data Preservation**: Ensure partial scan data is saved if possible

**Unsaved Data Handling:**
- **Pending Data Detection**: Check for unsaved test results or configurations
- **Save Prompts**: Offer user option to save data before logout
- **Auto-Save**: Automatically save critical data and user preferences
- **Data Integrity**: Ensure no data loss during logout process

### Session Security

**Session Invalidation:**
- **Token Revocation**: Immediately invalidate session token in database
- **Security Cleanup**: Clear all session-related authentication data
- **Audit Logging**: Record logout event with timestamp and reason
- **Memory Cleanup**: Clear sensitive data from system memory

**Multi-User Security:**
- **User Isolation**: Ensure no user data accessible after logout
- **Clean State**: Reset system to neutral state for next user
- **Permission Reset**: Clear all user-specific permissions and access levels
- **Cache Clearing**: Remove user-specific cached data and preferences

### Error Handling

**Logout Failures:**
- **Network Issues**: Handle communication failures with MW API
- **Database Errors**: Manage database connection or update failures
- **Active Scan Conflicts**: Handle situations where scan cannot be safely stopped
- **Partial Cleanup**: Ensure security even if some cleanup operations fail

**Recovery Actions:**
- **Force Logout**: Option to force logout even with active operations
- **Emergency Cleanup**: Fallback procedures for critical security cleanup
- **User Notification**: Clear communication about logout status and any issues
- **Administrative Override**: Admin capability to force user logout if needed

### User Experience Considerations

**Confirmation Flow:**
- **Optional Confirmation**: Configurable logout confirmation dialog
- **Context-Aware**: Different confirmation based on current system state
- **Quick Access**: Easy logout access from menu bar for convenience
- **Clear Feedback**: Visual confirmation that logout was successful

**Transition Experience:**
- **Smooth Transition**: Clean UI transition from main interface to login screen
- **Loading Indicators**: Show progress during logout cleanup process
- **Error Messages**: Clear communication if logout encounters issues
- **Next User Ready**: System immediately ready for next user login

### Key Design Decisions
- **Safety First**: Protect against data loss and unsafe scan termination
- **Security Focus**: Complete session invalidation and data cleanup
- **User Experience**: Smooth logout flow with appropriate confirmations
- **Audit Compliance**: Comprehensive logging for security and compliance
- **Multi-User Support**: Clean state transition for shared device usage
- **Error Resilience**: Robust handling of logout failures and edge cases

### Success Criteria
- User session properly invalidated and logged in database
- All user-specific data cleared from system memory and cache
- UI successfully transitions from main interface to login screen
- Active scans safely terminated with appropriate user warnings
- Unsaved data protected through save prompts or auto-save
- Session token revoked and security cleanup completed
- Audit trail entry created for logout event
- System ready for immediate use by next user

---

## Use Case 3: Shutdown/Sleep

### Overview
System power management that safely powers down or puts the XRF analyzer into sleep mode. System ensures proper shutdown sequence of all hardware components, data protection, and appropriate power state transition based on user selection (shutdown vs. sleep).

### Sequence Flow

```
Actor: User
UI Layer: User Interface
MW API: Middleware API
HW API: Hardware API
DB: Database
```

**Step-by-Step API Flow:**

### Power Management Initiation

1. **Power Menu Access**
   ```
   User -> UI: Navigate to menu bar/power options
   UI: Display power management options (Shutdown, Sleep)
   ```

2. **Power Action Selection**
   ```
   User -> UI: Press Shutdown button OR Sleep button
   ```

3. **Pre-Shutdown Safety Confirmation**
   ```
   UI: Display confirmation dialog based on action selected
   UI: "Are you sure you want to shutdown/sleep the system?"
   User -> UI: Confirm power action
   ```

4. **System State Validation**
   ```
   UI: Check for active scan operations
   UI: Check for unsaved data or pending operations
   
   If active scan in progress:
     UI: Display warning "Scan in progress. Shutdown will terminate measurement."
     User -> UI: Confirm or cancel shutdown
   
   If unsaved data exists:
     UI: Prompt to save critical data before shutdown
   ```

5. **Power Management Request**
   ```
   UI -> MW API: POST /api/power/shutdown (power_action: shutdown/sleep) (async)
   ```

### Hardware Shutdown Sequence

6. **Pre-Shutdown System Preparation**
   ```
   MW API: Validate system ready for power down
   MW API: Stop all active measurement processes
   MW API: Save system state and user session information
   MW API: Prepare for graceful hardware shutdown
   ```

7. **Hardware Component Shutdown**
   ```
   MW API -> HW API: shutdownXRayComponents() (sync)
   HW API: Safely shut down X-ray tube
   HW API: Turn off DPP (Digital Pulse Processor)
   HW API: Disable radiation LED
   HW API: Power down detector systems
   HW API -> MW API: Return hardware_shutdown_status (SUCCESS/FAILED)
   ```

8. **System Data Protection**
   ```
   MW API: Save current system configuration
   MW API -> DB: saveSystemState(shutdown_timestamp, power_action, user_session)
   MW API: Flush any pending database transactions
   MW API: Close database connections safely
   MW API -> DB: finalizeShutdownOperations()
   ```

9. **Power State Transition**
   ```
   If Shutdown selected:
     MW API -> HW API: performSystemShutdown() (sync)
     HW API: Power down all system components
     HW API: Initiate complete system shutdown
   
   If Sleep selected:
     MW API -> HW API: enterSleepMode() (sync)
     HW API: Power down non-essential components
     HW API: Maintain minimal power for wake capability
     MW API -> UI: initiateScreenSleep()
     UI: Turn off display while maintaining system responsiveness
   ```

10. **Shutdown Completion**
    ```
    MW API: Log power management event
    MW API -> UI: Return power_action_status (final status before shutdown/sleep)
    
    If Shutdown:
      System: Complete power down
    
    If Sleep:
      UI: Display sleep mode indicator
      System: Enter low-power state with wake capability
    ```

### API Sequence Diagram

```
User    UI       MW API    HW API    DB       System
 |       |         |         |       |         |
 |------>|         |         |       |         |  1. Press Shutdown/Sleep button
 |       | [confirmation]    |       |         |  2. Confirm power action
 |       | [safety checks]   |       |         |  3. Check active scans/data
 |------>|         |         |       |         |  4. Confirm after safety check
 |       |-------->|         |       |         |  5. POST /api/power/shutdown
 |       |         | [prep]  |       |         |  6. Prepare for shutdown
 |       |         |-------->|       |         |  7. shutdownXRayComponents()
 |       |         |<--------|       |         |  8. Return hardware_shutdown_status
 |       |         |---------------->|         |  9. saveSystemState()
 |       |         |-------->|       |         | 10. performSystemShutdown/Sleep()
 |       |<--------|         |       |         | 11. Return power_action_status
 |       |         |         |       |         | 12. Complete shutdown/sleep
 |       |         |         |       |         |
```

### Power Management Modes

**Shutdown Mode:**
- **Complete Power Down**: All system components powered off
- **Cold Boot Required**: Full startup sequence needed for next use
- **Maximum Power Savings**: Zero power consumption
- **Data Persistence**: All data saved to non-volatile storage

**Sleep Mode:**
- **Low Power State**: Essential components maintain minimal power
- **Quick Resume**: Fast wake-up capability for immediate use
- **Display Off**: Screen powered down while system remains responsive
- **Wake Triggers**: Touch, button press, or scheduled wake events

### Safety and Data Protection

**Hardware Safety:**
- **Radiation Safety**: X-ray tube safely powered down before shutdown
- **Component Protection**: Graceful shutdown prevents hardware damage
- **Thermal Management**: Allow components to cool down safely
- **Power Sequencing**: Proper order of component shutdown

**Data Protection:**
- **Unsaved Data**: Prompt user to save critical data before shutdown
- **System State**: Current configuration and session data preserved
- **Database Integrity**: All transactions completed before shutdown
- **Configuration Backup**: System settings saved for next startup

### Wake-Up from Sleep (Sleep Mode Only)

**Wake Triggers:**
- **Touch Screen**: Touch display to wake system
- **Hardware Buttons**: Physical button press to resume
- **Scheduled Wake**: Timer-based automatic wake for maintenance
- **External Events**: Network activity or external device connection

**Wake Sequence:**
```
User/System -> HW API: Wake trigger detected
HW API: Power up essential components
HW API -> MW API: systemWakeEvent()
MW API: Restore system state from sleep
MW API -> UI: resumeFromSleep()
UI: Turn on display and restore interface
```

### Error Handling

**Shutdown Failures:**
- **Hardware Errors**: Component fails to shut down properly
- **Data Save Failures**: Database or file system errors during save
- **Active Scan Conflicts**: Cannot safely terminate active measurements
- **Power System Issues**: Hardware power management failures

**Recovery Actions:**
- **Force Shutdown**: Emergency shutdown despite component failures
- **Partial Shutdown**: Shut down successfully completed components
- **Error Logging**: Record shutdown failures for maintenance
- **Safe Mode**: Attempt minimal safe configuration before shutdown

### System Recovery

**Startup After Shutdown:**
- **Cold Boot**: Full system initialization and hardware startup
- **State Restoration**: Reload saved system configuration
- **User Session**: Restore or require new user authentication
- **Hardware Verification**: Confirm all components operational

**Resume from Sleep:**
- **Quick Wake**: Immediate system responsiveness
- **State Continuity**: Maintain system state and user session
- **Display Restoration**: Return to previous screen state
- **Component Activation**: Restore active hardware components

### Key Design Decisions
- **Safety First**: Radiation safety and hardware protection prioritized
- **Data Protection**: Comprehensive data saving before power state change
- **User Choice**: Clear distinction between shutdown and sleep modes
- **Graceful Termination**: Proper component shutdown sequence
- **Wake Capability**: Sleep mode maintains quick resume functionality
- **Error Resilience**: Robust handling of shutdown failures

### Success Criteria
- All hardware components safely powered down in correct sequence
- X-ray tube and DPP properly shut down with radiation safety maintained
- System state and user data saved before power state transition
- Unsaved data protected through user prompts and auto-save
- Power action completed successfully (shutdown or sleep)
- System ready for proper startup or wake sequence
- Comprehensive logging of power management events
- No hardware damage or data loss during power transitions

---

## Use Case 4: Brightness Control

### Overview
Display brightness adjustment through settings menu slider control. System provides user interface for adjusting screen brightness levels and applies the selected brightness setting to the display hardware for optimal viewing in different lighting conditions.

### Sequence Flow

```
Actor: User
UI Layer: User Interface
MW API: Middleware API  
HW API: Hardware API
```

**Step-by-Step API Flow:**

### Brightness Settings Access

1. **Settings Menu Navigation**
   ```
   User -> UI: Navigate to Settings menu
   UI: Display settings interface with various configuration options
   UI: Show Display/Brightness section with current brightness level
   ```

2. **Brightness Slider Display**
   ```
   UI: Display brightness slider control (range: 0-100%)
   UI: Show current brightness level on slider
   UI: Display brightness percentage value
   ```

### Real-Time Brightness Adjustment

3. **User Slider Interaction**
   ```
   User -> UI: Moves brightness slider to desired level
   UI: Capture slider position change event
   UI: Calculate new brightness percentage (0-100%)
   ```

4. **Immediate Brightness Update**
   ```
   UI -> MW API: PUT /api/display/brightness (brightness_level) (async)
   ```

5. **Display Hardware Control**
   ```
   MW API -> HW API: setBrightness(brightness_percentage) (sync)
   HW API: Adjust display backlight intensity
   HW API: Apply brightness setting to screen hardware
   HW API: Validate brightness level applied successfully
   HW API -> MW API: Return brightness_status (SUCCESS/FAILED)
   ```

6. **Brightness Confirmation**
   ```
   MW API -> UI: Return brightness_update_status (success/failure, current_level)
   UI: Update slider visual feedback if needed
   UI: Display current brightness percentage value
   ```

7. **Setting Persistence**
   ```
   MW API: Save brightness preference to user settings
   MW API: Store brightness level for system restoration after power cycles
   ```

### API Sequence Diagram

```
User    UI       MW API    HW API
 |       |         |         |
 |------>|         |         |  1. Navigate to Settings/Brightness
 |       | [show slider]     |  2. Display brightness slider
 |------>|         |         |  3. Move slider to new level
 |       |-------->|         |  4. PUT /api/display/brightness
 |       |         |-------->|  5. setBrightness() (sync)
 |       |         |         | [adjust display] 6. Apply to hardware
 |       |         |<--------|  7. Return brightness_status
 |       |<--------|         |  8. Return brightness_update_status
 |       | [update display]  |  9. Show new brightness level
 |       |         | [save]  | 10. Persist brightness setting
 |       |         |         |
```

### Brightness Control Features

**Brightness Range:**
- **Minimum**: 10% (prevents completely dark screen)
- **Maximum**: 100% (full display brightness)
- **Increments**: 5% or 10% increments for precise control
- **Default**: 50% or ambient-adjusted default level

**Real-Time Adjustment:**
- **Live Preview**: Brightness changes immediately as user moves slider
- **Smooth Transition**: Gradual brightness changes, not abrupt jumps
- **Visual Feedback**: Slider position and percentage display update instantly
- **Hardware Sync**: Display brightness matches slider position precisely

### User Experience Design

**Slider Interface:**
- **Visual Slider**: Horizontal brightness slider with clear position indicator
- **Percentage Display**: Numeric display of current brightness level (e.g., "75%")
- **Preview Effect**: Screen brightness changes immediately as slider moves
- **Reset Option**: Button to restore default brightness level

**Environmental Adaptation:**
- **Auto-Adjustment**: Optional automatic brightness based on ambient light
- **Manual Override**: User can override auto settings with manual control
- **Memory**: System remembers last user-set brightness level
- **Power Saving**: Lower brightness options for extended battery life

### System Integration

**Settings Persistence:**
- **User Preferences**: Brightness level saved per user account
- **System Defaults**: Default brightness for new users or system reset
- **Power Cycle Memory**: Brightness setting restored after shutdown/restart
- **Profile Integration**: Brightness included in user preference profiles

**Hardware Compatibility:**
- **Display Driver**: Direct control of display backlight hardware
- **Power Management**: Integration with system power management
- **Hardware Limits**: Respect hardware minimum/maximum brightness capabilities
- **Error Handling**: Graceful handling of hardware brightness control failures

### Safety Considerations

**Display Safety:**
- **Minimum Brightness**: Prevent screen from becoming completely black
- **Maximum Safety**: Prevent excessive brightness that could damage display
- **User Comfort**: Appropriate brightness ranges for extended use
- **Eye Strain**: Smooth transitions to prevent eye strain

**System Safety:**
- **Power Management**: Brightness adjustment coordinated with power systems
- **Hardware Protection**: Prevent brightness settings that could damage hardware
- **Emergency Override**: Admin capability to reset brightness if needed
- **Diagnostic Mode**: Special brightness modes for system diagnostics

### Key Design Decisions
- **Real-Time Updates**: Brightness changes immediately as user adjusts slider
- **Sync Hardware Control**: MW API waits for hardware confirmation before responding
- **User Preference Storage**: Brightness level persisted for user experience continuity
- **Safety Boundaries**: Minimum and maximum limits prevent unusable or harmful settings
- **Smooth UI Interaction**: Responsive slider with immediate visual feedback
- **Power Integration**: Brightness control integrated with overall system power management

### Success Criteria
- User can easily adjust brightness using intuitive slider interface
- Screen brightness changes immediately and smoothly as slider moves
- Brightness setting persists across power cycles and user sessions
- Hardware brightness control responds accurately to software commands
- Brightness percentage display matches actual screen brightness level
- System maintains brightness preferences per user account
- Brightness control integrates properly with power management features

---

## Use Case 5: Language Settings

### Overview
System localization control that allows users to select their preferred language for the device interface. System applies comprehensive language changes to all UI elements, messages, labels, and data formats while maintaining user preferences and providing immediate interface translation.

### Sequence Flow

```
Actor: User
UI Layer: User Interface
MW API: Middleware API  
DB: Database
```

**Step-by-Step API Flow:**

### Language Settings Access

1. **Settings Menu Navigation**
   ```
   User -> UI: Navigate to Settings menu
   UI: Display settings interface with various configuration options
   UI: Show Language/Localization section with current language setting
   ```

2. **Language Selection Display**
   ```
   UI: Display language selection button/dropdown
   UI: Show current active language (e.g., "English", "Español", "Français")
   UI: Display available language options based on system capabilities
   ```

### Language Selection Process

3. **Language Menu Access**
   ```
   User -> UI: Presses Language button/dropdown
   UI: Display available language list with language names in native script
   UI: Highlight current active language selection
   UI: Show language options (English, Spanish, French, German, Chinese, etc.)
   ```

4. **Language Selection**
   ```
   User -> UI: Selects desired language from list
   UI: Capture language selection event
   UI: Display confirmation dialog in both current and new language
   ```

5. **Language Change Request**
   ```
   User -> UI: Confirms language change
   UI -> MW API: PUT /api/system/language (language_code, user_id) (async)
   ```

### Language Application Process

6. **Language Validation and Loading**
   ```
   MW API: Validate selected language code (ISO 639-1 standard)
   MW API: Check language pack availability and completeness
   MW API: Load language resource files and translation data
   MW API: Validate translation completeness for all UI components
   ```

7. **System Language Configuration**
   ```
   MW API: Update system locale settings
   MW API: Configure number formatting (decimal separators, thousands)
   MW API: Set date/time formatting for selected language region
   MW API: Update measurement unit displays based on regional preferences
   ```

8. **Language Application Response**
   ```
   MW API -> UI: Return language_change_status (success/failure, language_info)
   ```

9. **UI Language Update**
   ```
   UI: Apply new language translations to all interface elements
   UI: Update menu labels, button text, dialog messages
   UI: Refresh measurement displays with localized units
   UI: Update date/time formats throughout interface
   UI: Reload current screen with new language
   ```

10. **User Preference Storage**
    ```
    MW API -> DB: saveUserLanguagePreference(user_id, language_code, timestamp)
    MW API: Store language setting as system default if admin user
    MW API: Update user profile with language preference
    ```

11. **Language Change Completion**
    ```
    UI: Display language change confirmation in new language
    UI: Update language button to show new active language
    UI: Show completion notification in selected language
    ```

### API Sequence Diagram

```
User    UI       MW API            DB
 |       |         |                |
 |------>|         |                |  1. Navigate to Settings/Language
 |       | [show current lang]      |  2. Display language button
 |------>|         |                |  3. Press Language button
 |       | [show language list]     |  4. Display available languages
 |------>|         |                |  5. Select desired language
 |       | [confirmation dialog]    |  6. Confirm language change
 |------>|         |                |  7. Confirm change
 |       |-------->|                |  8. PUT /api/system/language
 |       |         | [validate & load] |  9. Load language resources
 |       |         | [configure system] | 10. Update system locale
 |       |<--------|                | 11. Return language_change_status
 |       | [apply translations]     | 12. Update all UI elements
 |       |         |--------------->| 13. saveUserLanguagePreference()
 |       | [show completion]        | 14. Display confirmation in new language
 |       |         |                |
```

### Supported Languages

**Primary Languages:**
- **English (US)**: Default language with complete feature set
- **Spanish (ES)**: Full localization including regional measurements
- **French (FR)**: Complete translation with European formatting
- **German (DE)**: Full localization with German technical terminology
- **Chinese (CN)**: Simplified Chinese with appropriate character encoding
- **Japanese (JP)**: Complete Kanji/Hiragana support with proper formatting

**Regional Considerations:**
- **Number Formats**: Decimal separators, thousands separators by region
- **Date Formats**: DD/MM/YYYY vs MM/DD/YYYY vs YYYY-MM-DD
- **Measurement Units**: Metric vs Imperial based on regional preferences
- **Currency Symbols**: Regional currency displays where applicable

### Localization Components

**UI Text Translation:**
- **Menu Labels**: All navigation and menu items
- **Button Text**: Action buttons, confirmations, cancellations
- **Dialog Messages**: Warnings, errors, confirmations, help text
- **Status Indicators**: System status, measurement states, progress messages
- **Form Labels**: Input field labels, dropdown options, validation messages

**Data Formatting:**
- **Numbers**: Decimal places, separators, scientific notation
- **Dates/Times**: Format patterns, month names, day names
- **Measurements**: Unit displays, precision, engineering notation
- **Chemistry Data**: Element symbols, concentration units, uncertainty formats

### System Integration

**User Preferences:**
- **Per-User Settings**: Individual language preferences by user account
- **System Defaults**: Default language for login screen and new users
- **Admin Override**: Administrative capability to set system-wide language
- **Persistence**: Language setting maintained across power cycles and sessions

**Application Integration:**
- **Real-Time Updates**: All interface elements update immediately
- **Screen Refresh**: Current screen reloads with new language
- **Data Display**: Measurement results displayed in localized format
- **Export Formats**: Report templates and exports use selected language

### Technical Implementation

**Language Resources:**
- **Translation Files**: JSON/XML files with key-value translation pairs
- **Resource Loading**: Dynamic loading of language packs from storage
- **Fallback Mechanism**: Default to English if translation missing
- **Encoding Support**: UTF-8 encoding for international character sets

**Performance Considerations:**
- **Resource Caching**: Language resources cached in memory for performance
- **Lazy Loading**: Load translation resources only when needed
- **Memory Management**: Efficient storage and retrieval of translation data
- **Update Mechanism**: Support for language pack updates and additions

### Error Handling

**Language Change Failures:**
- **Invalid Language Code**: Handle unsupported or malformed language codes
- **Missing Resources**: Graceful handling when translation files unavailable
- **Partial Translations**: Display mixed language when translations incomplete
- **System Errors**: Handle memory or file system issues during language change

**Recovery Actions:**
- **Fallback Language**: Revert to previous language if new selection fails
- **English Default**: Fall back to English for missing translations
- **Error Messages**: Clear error communication in user's current language
- **Retry Mechanism**: Option to retry language change after failure

### User Experience Design

**Language Selection Interface:**
- **Native Display**: Language names shown in their native script
- **Visual Indicators**: Flags or cultural icons to aid recognition
- **Current Selection**: Clear indication of currently active language
- **Preview Mode**: Optional preview of interface in selected language

**Transition Experience:**
- **Smooth Updates**: Interface elements update cleanly without flicker
- **Progress Indicators**: Show language change progress for user feedback
- **Confirmation**: Clear confirmation that language change was successful
- **Context Preservation**: Return to same screen location after language change

### Key Design Decisions
- **Immediate Application**: Language changes apply instantly across all UI elements
- **User-Specific Settings**: Language preferences stored per user account
- **Comprehensive Translation**: All user-facing text translated, not just menus
- **Regional Formatting**: Numbers, dates, units formatted according to language region
- **Fallback Strategy**: English fallback for missing translations maintains usability
- **Resource Efficiency**: Language packs loaded on-demand with memory optimization

### Success Criteria
- User can easily select desired language from clear, accessible interface
- All UI elements immediately update to display selected language
- Number, date, and measurement formats apply regional standards correctly
- Language preference persists across user sessions and power cycles
- System maintains full functionality in all supported languages
- Translation quality maintains technical accuracy for XRF terminology
- Language change process completes smoothly without interface disruption

---

## Use Case 6: Firmware Update

### Overview
System update management that provides both automatic periodic update checks and manual update initiation through settings menu. System manages software updates for the main application and firmware updates for hardware components, ensuring secure download, validation, and installation of updates while maintaining system safety and data integrity.

### Sequence Flow

```
Actor: User/System (Automatic)
UI Layer: User Interface
MW API: Middleware API  
HW API: Hardware API
DB: Database
Update Server: External Update Server
```

**Step-by-Step API Flow:**

### Automatic Update Check (Periodic - On User Login)

1. **Login-Triggered Update Check**
   ```
   System: User login completed successfully
   MW API: Initiate automatic update check process
   MW API: Check last update check timestamp
   ```

2. **Update Server Communication**
   ```
   MW API -> Update Server: GET /api/updates/check (device_id, current_versions) (async)
   Update Server: Compare device versions against latest available
   Update Server: Check for software updates (MW API, UI, databases)
   Update Server: Check for firmware updates (HW API, DPP, X-ray controller)
   Update Server -> MW API: Return update_availability (available_updates, versions, sizes)
   ```

3. **Update Notification Display**
   ```
   MW API -> UI: displayUpdateNotification(update_info) (async)
   UI: Show update notification banner on home page
   UI: Display available update count and importance level
   UI: Provide "Update Now" and "Later" options
   ```

### Manual Update Check (Settings Menu)

4. **Settings Menu Access**
   ```
   User -> UI: Navigate to Settings menu
   UI: Display settings interface with System/Updates section
   UI: Show "Check for Updates" button with last check timestamp
   ```

5. **Manual Update Request**
   ```
   User -> UI: Press "Check for Updates" button
   UI -> MW API: POST /api/updates/manual-check (async)
   UI: Display "Checking for updates..." progress indicator
   ```

6. **Update Discovery and Validation**
   ```
   MW API -> Update Server: GET /api/updates/check (device_id, current_versions) (async)
   Update Server -> MW API: Return update_availability (available_updates, details)
   MW API: Validate update authenticity and digital signatures
   MW API: Check update compatibility with current system
   MW API: Calculate download sizes and estimated installation time
   ```

### Update Installation Process

7. **Update Confirmation**
   ```
   MW API -> UI: displayAvailableUpdates(update_list, details) (async)
   UI: Show update details (versions, sizes, changes, importance)
   UI: Display installation time estimate and system requirements
   User -> UI: Confirms update installation
   ```

8. **Pre-Update Safety Validation**
   ```
   MW API: Check system state (must not be actively scanning)
   MW API: Verify sufficient storage space for updates
   MW API: Check battery level (if portable device)
   MW API: Create system backup/restore point
   MW API -> DB: backupSystemConfiguration()
   ```

9. **Update Download Phase**
   ```
   MW API -> Update Server: GET /api/updates/download (update_ids) (async)
   MW API: Download update packages with progress tracking
   MW API -> UI: updateDownloadProgress(percent_complete) (async)
   UI: Display download progress bar and current operation
   MW API: Validate downloaded packages (checksums, signatures)
   ```

### Software Update Application

10. **Software Update Installation**
    ```
    MW API: Begin software update installation sequence
    MW API: Stop non-essential system services
    MW API: Apply UI application updates
    MW API: Apply MW API service updates
    MW API: Update database schemas if required
    MW API: Update configuration files and settings
    ```

### Firmware Update Process (If Available)

11. **Hardware Firmware Update**
    ```
    MW API: Check if firmware updates are included in package
    MW API -> HW API: beginFirmwareUpdate(firmware_image_data) (sync)
    HW API: Validate firmware image authenticity and compatibility
    HW API: Create firmware backup for rollback capability
    HW API: Apply firmware update to hardware components
    HW API: Update X-ray controller firmware if included
    HW API: Update DPP firmware if included
    HW API: Verify firmware installation success
    HW API -> MW API: Return firmware_update_status (SUCCESS/FAILED)
    ```

12. **System Restart and Validation**
    ```
    MW API: Complete update installation
    MW API: Schedule system restart for update activation
    MW API -> UI: displayUpdateCompletion(restart_required) (async)
    UI: Show update completion message and restart notification
    User -> UI: Confirms system restart
    MW API: Initiate controlled system restart
    ```

13. **Post-Update Verification**
    ```
    System: Restart completed, system initialization
    MW API: Verify all updates applied successfully
    MW API: Check system functionality and hardware communication
    MW API -> HW API: verifyFirmwareVersions()
    HW API -> MW API: Return current_firmware_versions
    MW API: Compare versions against expected update results
    MW API -> DB: logUpdateEvent(update_results, versions, timestamp)
    ```

### API Sequence Diagram

```
User/Sys UI     MW API    Update Server  HW API    DB
   |      |       |           |           |       |
   |      | [login trigger]   |           |       |  1. Auto check on login
   |      |       |---------->|           |       |  2. GET /api/updates/check
   |      |       |<----------|           |       |  3. Return update_availability
   |      |<------|           |           |       |  4. displayUpdateNotification()
   |      | [show banner]     |           |       |  5. Update notification displayed
   |----->|       |           |           |       |  6. Press "Check Updates" (manual)
   |      |------>|           |           |       |  7. POST /api/updates/manual-check
   |      |       |---------->|           |       |  8. GET /api/updates/check
   |      |       |<----------|           |       |  9. Return available updates
   |      |<------|           |           |       | 10. displayAvailableUpdates()
   |----->|       |           |           |       | 11. Confirm update installation
   |      |       | [safety checks]       |       | 12. Validate system ready
   |      |       |---------------->      |       | 13. backupSystemConfiguration()
   |      |       |---------->|           |       | 14. GET /api/updates/download
   |      |<------|           |           |       | 15. updateDownloadProgress()
   |      |       | [install SW]          |       | 16. Apply software updates
   |      |       |------------------>    |       | 17. beginFirmwareUpdate() [if needed]
   |      |       |                   | [update] | 18. Apply firmware to hardware
   |      |       |<------------------    |       | 19. Return firmware_update_status
   |      |<------|           |           |       | 20. displayUpdateCompletion()
   |      | [restart system]  |           |       | 21. System restart
   |      |       | [verify]  |           |       | 22. Post-update verification
   |      |       |---------------->      |       | 23. verifyFirmwareVersions()
   |      |       |------------------------->     | 24. logUpdateEvent()
   |      |       |           |           |       |
```

### Update Types and Components

**Software Updates:**
- **UI Application**: User interface updates, new features, bug fixes
- **Middleware API**: Core system logic, algorithms, security patches
- **Database Schema**: Table structure updates, index optimizations
- **Configuration Files**: System settings, calibration parameters
- **Language Packs**: Translation updates, new language support

**Firmware Updates:**
- **X-ray Controller**: Tube control firmware, safety systems
- **DPP Firmware**: Digital pulse processor updates, signal processing
- **System Bootloader**: Low-level system initialization code
- **Hardware Drivers**: Device-specific hardware communication updates
- **Safety Systems**: Emergency stop and radiation safety firmware

### Update Security and Validation

**Security Measures:**
- **Digital Signatures**: All updates cryptographically signed by manufacturer
- **Checksum Validation**: Package integrity verified before installation
- **Authentication**: Update server authentication prevents malicious updates
- **Rollback Capability**: Automatic rollback if update installation fails

**Compatibility Checks:**
- **Version Compatibility**: Ensure updates compatible with current system
- **Hardware Requirements**: Verify hardware supports updated features
- **Dependency Validation**: Check all required components present
- **Configuration Migration**: Update settings and calibrations appropriately

### Update Scheduling and Management

**Automatic Update Checks:**
- **Login Triggers**: Check for updates every user login
- **Scheduled Checks**: Daily/weekly automatic update discovery
- **Critical Updates**: Immediate notification for security updates
- **Update Deferrals**: Allow users to postpone non-critical updates

**Update Installation Timing:**
- **User-Initiated**: Updates installed when user requests
- **Scheduled Installation**: Install during off-hours if configured
- **Maintenance Windows**: Coordinate with system maintenance schedules
- **Emergency Updates**: Critical security updates with immediate installation

### Error Handling and Recovery

**Update Failures:**
- **Download Failures**: Network issues, corrupted packages, insufficient space
- **Installation Failures**: Software conflicts, hardware compatibility issues
- **Firmware Failures**: Hardware update failures, communication errors
- **Verification Failures**: Post-update system validation failures

**Recovery Procedures:**
- **Automatic Rollback**: Revert to previous versions if update fails
- **Safe Mode Boot**: Boot into recovery mode if system fails to start
- **Manual Recovery**: Administrative tools for manual system recovery
- **Factory Reset**: Last resort recovery to known good state

### User Experience Design

**Update Notifications:**
- **Non-Intrusive Banners**: Update notifications that don't block workflow
- **Clear Information**: Update details, benefits, estimated installation time
- **User Control**: Options to install now, schedule later, or skip
- **Progress Feedback**: Clear progress indicators during installation

**Installation Experience:**
- **Progress Tracking**: Real-time feedback on download and installation progress
- **Time Estimates**: Accurate estimates of installation duration
- **System Status**: Clear communication of system state during updates
- **Completion Confirmation**: Success notification with update summary

### Key Design Decisions
- **Automatic Discovery**: Periodic update checks without user intervention
- **User Control**: Users can manually check and control update installation
- **Comprehensive Updates**: Both software and firmware updates managed
- **Safety First**: Updates blocked during active scans or unsafe conditions
- **Secure Process**: Digital signature validation and rollback capabilities
- **Progress Transparency**: Clear feedback throughout entire update process

### Success Criteria
- System automatically discovers and notifies users of available updates
- Users can manually check for updates through settings interface
- Software updates install successfully without data loss or corruption
- Firmware updates apply correctly to hardware components when needed
- Update process provides clear progress feedback and completion status
- System remains secure through signature validation and authentication
- Failed updates rollback automatically to maintain system stability
- Update history and results properly logged for audit and troubleshooting

---

## Use Case 7: Data Sync to USB/SD/Cloud

### Overview
Data synchronization system that enables users to backup and transfer scan data to external storage devices or cloud services. System provides flexible data selection, multiple destination options (USB, SD card, cloud), and secure transfer protocols while maintaining data integrity and providing comprehensive progress feedback.

### Sequence Flow

```
Actor: User
UI Layer: User Interface
MW API: Middleware API  
HW API: Hardware API
DB: Database
Cloud Service: External Cloud Storage
```

**Step-by-Step API Flow:**

### Data Sync Settings Access

1. **Settings Menu Navigation**
   ```
   User -> UI: Navigate to Settings menu
   UI: Display settings interface with Data/Sync section
   UI: Show "Data Sync" button with last sync status and timestamp
   ```

2. **Data Sync Interface Display**
   ```
   User -> UI: Press "Data Sync" button
   UI -> MW API: GET /api/sync/status (async)
   MW API: Check available storage devices (USB, SD card)
   MW API: Check cloud service connectivity and authentication
   MW API -> UI: Return sync_status (available_destinations, last_sync_info)
   UI: Display sync interface with destination options
   ```

### Data Selection Process

3. **Available Data Display**
   ```
   UI -> MW API: GET /api/sync/scan-data (date_range) (async)
   MW API -> DB: getAvailableScanData(date_filter, user_permissions)
   DB -> MW API: Return scan_list (session_ids, dates, sizes, sync_status)
   MW API -> UI: Return available_scans (scan_summaries, total_size)
   UI: Display selectable list of scans with checkboxes
   UI: Show scan details (date, mode, size, chemistry results)
   UI: Display total selected data size and estimated transfer time
   ```

4. **Scan Selection**
   ```
   User -> UI: Selects desired scans using checkboxes or date range
   UI: Update selection summary (count, total size, estimated time)
   UI: Enable destination selection and sync options
   ```

5. **Destination Selection**
   ```
   User -> UI: Selects sync destination (USB, SD Card, or Cloud)
   UI: Display destination-specific options and settings
   UI: Show available space on selected destination
   ```

### USB Sync Process

6. **USB Sync Configuration** (If USB selected)
   ```
   User -> UI: Configures USB sync settings (file format, folder structure)
   UI: Display USB device information and available space
   User -> UI: Confirms USB sync initiation
   UI -> MW API: POST /api/sync/start-usb (selected_scans, sync_options) (async)
   ```

7. **USB Data Preparation and Transfer**
   ```
   MW API -> DB: retrieveScanData(selected_session_ids, include_metadata)
   DB -> MW API: Return complete_scan_data (spectrum, chemistry, images, notes)
   MW API: Format data according to user preferences (CSV, JSON, PDF)
   MW API: Package data files with proper folder structure
   MW API -> HW API: initiateUSBTransfer(data_package, destination_path) (sync)
   HW API: Establish SFTP connection to USB device
   HW API: Create folder structure on USB destination
   HW API: Transfer files using secure file transfer protocol
   HW API -> MW API: Return usb_transfer_progress (percent_complete, current_file)
   ```

### SD Card Sync Process

8. **SD Card Sync Configuration** (If SD Card selected)
   ```
   User -> UI: Configures SD card sync settings (file organization, formats)
   UI: Display SD card information and available space
   User -> UI: Confirms SD card sync initiation
   UI -> MW API: POST /api/sync/start-sd (selected_scans, sync_options) (async)
   ```

9. **SD Card Data Transfer**
   ```
   MW API -> DB: retrieveScanData(selected_session_ids, include_metadata)
   DB -> MW API: Return complete_scan_data (spectrum, chemistry, images, notes)
   MW API: Format data according to user preferences
   MW API -> HW API: initiateSDTransfer(data_package, folder_structure) (sync)
   HW API: Access SD card file system
   HW API: Create organized folder structure (by date, mode, user)
   HW API: Copy data files to SD card destination folders
   HW API: Verify file integrity after transfer
   HW API -> MW API: Return sd_transfer_progress (percent_complete, files_transferred)
   ```

### Cloud Sync Process

10. **Cloud Sync Configuration** (If Cloud selected)
    ```
    User -> UI: Selects cloud service and configures sync settings
    UI: Display cloud authentication status and available storage
    User -> UI: Confirms cloud sync initiation
    UI -> MW API: POST /api/sync/start-cloud (selected_scans, cloud_service, sync_options) (async)
    ```

11. **Cloud Data Upload**
    ```
    MW API -> DB: retrieveScanData(selected_session_ids, include_metadata)
    DB -> MW API: Return complete_scan_data (spectrum, chemistry, images, notes)
    MW API: Format data according to cloud service requirements
    MW API: Encrypt data for secure cloud transmission
    MW API -> Cloud Service: uploadScanData(encrypted_data_package) (async)
    Cloud Service: Store data with proper organization and metadata
    Cloud Service -> MW API: Return cloud_upload_progress (percent_complete, status)
    ```

### Progress Monitoring and Completion

12. **Real-Time Progress Updates**
    ```
    MW API -> UI: updateSyncProgress(destination, percent_complete, current_operation) (async)
    UI: Display real-time progress bar with current file/operation
    UI: Show transfer speed, estimated time remaining, files completed
    UI: Update sync status and allow user to cancel if needed
    ```

13. **Sync Completion and Verification**
    ```
    MW API: Complete data transfer to selected destination
    MW API: Verify data integrity and transfer success
    MW API -> DB: updateSyncStatus(session_ids, destination, sync_timestamp)
    MW API -> UI: displaySyncCompletion(sync_results, verification_status) (async)
    UI: Show sync completion message with transfer summary
    UI: Display successful transfers, any failures, total data synced
    UI: Update sync status indicators for completed scans
    ```

### API Sequence Diagram

```
User    UI       MW API    DB      HW API    Cloud Service
 |       |         |        |        |           |
 |------>|         |        |        |           |  1. Navigate to Data Sync
 |       |-------->|        |        |           |  2. GET /api/sync/status
 |       |<--------|        |        |           |  3. Return sync_status
 |       |-------->|        |        |           |  4. GET /api/sync/scan-data
 |       |         |------->|        |           |  5. getAvailableScanData()
 |       |         |<-------|        |           |  6. Return scan_list
 |       |<--------|        |        |           |  7. Return available_scans
 |       | [show scan list] |        |           |  8. Display selectable scans
 |------>|         |        |        |           |  9. Select scans & destination
 |       |-------->|        |        |           | 10. POST /api/sync/start-[destination]
 |       |         |------->|        |           | 11. retrieveScanData()
 |       |         |<-------|        |           | 12. Return complete_scan_data
 |       |         |--------------->|           | 13. initiateUSBTransfer() [USB path]
 |       |         |                | [transfer]| 14. Transfer via SFTP protocol
 |       |         |<---------------|           | 15. Return transfer_progress
 |       |         |------------------------->| 16. uploadScanData() [Cloud path]
 |       |         |<-------------------------| 17. Return upload_progress
 |       |<--------|        |        |           | 18. updateSyncProgress()
 |       | [show progress]  |        |           | 19. Display real-time progress
 |       |         |------->|        |           | 20. updateSyncStatus()
 |       |<--------|        |        |           | 21. displaySyncCompletion()
 |       | [show completion]|        |           | 22. Show sync results
 |       |         |        |        |           |
```

### Sync Destinations and Protocols

**USB Device Sync:**
- **Protocol**: Secure File Transfer Protocol (SFTP) for data integrity
- **File Structure**: Organized folders by date, test mode, or user preference
- **Formats**: Multiple export formats (CSV, JSON, PDF reports)
- **Verification**: File integrity checks after transfer completion

**SD Card Sync:**
- **Protocol**: Direct file system access with verification
- **Organization**: Hierarchical folder structure (Date/Mode/User)
- **Storage**: Local file copy with metadata preservation
- **Management**: Automatic folder creation and space management

**Cloud Sync:**
- **Services**: Support for major cloud providers (AWS, Azure, Google Cloud)
- **Security**: End-to-end encryption for data transmission
- **Authentication**: OAuth or API key authentication
- **Synchronization**: Incremental sync to avoid duplicate transfers

### Data Components and Formatting

**Scan Data Components:**
- **Spectrum Data**: Raw and processed spectrum arrays with calibration
- **Chemistry Results**: Element concentrations, uncertainties, detection limits
- **Metadata**: Test conditions, timestamps, operator information, GPS location
- **Images**: Sample photographs and visual documentation
- **Notes**: Text annotations and contextual information

**Export Format Options:**
- **CSV Format**: Tabular chemistry data for spreadsheet analysis
- **JSON Format**: Structured data for system integration
- **PDF Reports**: Professional formatted reports with charts
- **Raw Data**: Native format files for system backup/restore

### Storage Management

**Space Management:**
- **Size Calculation**: Accurate storage requirements before sync
- **Available Space**: Check destination capacity before transfer
- **Compression**: Optional data compression to reduce transfer size
- **Cleanup**: Automatic cleanup of temporary files after sync

**Sync Status Tracking:**
- **Individual Scan Status**: Track sync status per scan session
- **Destination Tracking**: Record which destinations have copies
- **Sync History**: Maintain log of all sync operations
- **Error Tracking**: Record and report failed sync attempts

### Security and Data Integrity

**Data Protection:**
- **Encryption**: Data encrypted during transmission to cloud services
- **Authentication**: Secure authentication for cloud and network destinations
- **Access Control**: User permission validation for data access
- **Audit Trail**: Complete logging of all sync activities

**Integrity Verification:**
- **Checksum Validation**: File integrity verification after transfer
- **Data Completeness**: Verify all selected data transferred successfully
- **Error Detection**: Identify and report any corrupted transfers
- **Retry Mechanism**: Automatic retry for failed transfers

### User Experience Design

**Selection Interface:**
- **Intuitive Selection**: Easy scan selection with preview information
- **Bulk Operations**: Select all, date ranges, or filtered selections
- **Progress Feedback**: Real-time transfer progress and status updates
- **Cancel Capability**: Allow users to cancel ongoing sync operations

**Status and Notifications:**
- **Sync Status**: Clear indicators of sync status for each scan
- **Completion Notifications**: Success/failure notifications with details
- **Error Messages**: Clear error reporting with resolution suggestions
- **History View**: View of previous sync operations and results

### Key Design Decisions
- **Multiple Destinations**: Support for USB, SD card, and cloud sync options
- **User Selection Control**: Allow users to choose specific scans to sync
- **Secure Protocols**: SFTP for USB, encryption for cloud transfers
- **Data Integrity**: Comprehensive verification and error handling
- **Progress Transparency**: Real-time feedback throughout sync process
- **Flexible Formatting**: Multiple export formats based on destination needs

### Success Criteria
- Users can easily select specific scans or date ranges for synchronization
- System successfully transfers data to USB devices using secure SFTP protocol
- SD card sync creates organized folder structure with complete data files
- Cloud sync provides encrypted, secure data backup to external services
- Real-time progress feedback keeps users informed during transfer process
- Data integrity verification ensures all transfers are complete and accurate
- Sync status tracking maintains history and prevents duplicate transfers
- Error handling provides clear feedback and recovery options for failed syncs

---

## Use Case 8: View System Health

### Overview
Comprehensive system monitoring interface that displays real-time health status of all hardware components and sensors in the XRF analyzer system. System provides visual dashboard with charts, status indicators, and diagnostic information to help users and technicians monitor system performance and identify potential issues before they affect measurements.

### Sequence Flow

```
Actor: User
UI Layer: User Interface
MW API: Middleware API  
HW API: Hardware API
DB: Database
```

**Step-by-Step API Flow:**

### System Health Page Access

1. **Settings Menu Navigation**
   ```
   User -> UI: Navigate to Settings menu
   UI: Display settings interface with System/Diagnostics section
   UI: Show "System Health" button with current system status indicator
   ```

2. **System Health Page Request**
   ```
   User -> UI: Press "System Health" button
   UI -> MW API: GET /api/system/health-status (async)
   UI: Display "Loading system health..." progress indicator
   ```

### Hardware Status Collection

3. **Hardware Polling Initiation**
   ```
   MW API -> HW API: pollAllDevices() (sync)
   HW API: Begin comprehensive hardware status collection
   ```

4. **X-ray Tube Status Collection**
   ```
   HW API: Poll X-ray tube controller
   HW API: Read tube voltage, current, temperature, filament status
   HW API: Check tube operational hours, high voltage status
   HW API: Monitor tube safety interlocks and fault conditions
   HW API: Collect tube performance metrics and aging indicators
   ```

5. **DPP (Digital Pulse Processor) Status Collection**
   ```
   HW API: Poll DPP hardware
   HW API: Read detector temperature, high voltage status
   HW API: Check spectrum processing performance, count rates
   HW API: Monitor DPP firmware status and communication health
   HW API: Collect detector performance metrics and stability data
   ```

6. **System Sensor Status Collection**
   ```
   HW API: Poll environmental sensors
   HW API: Read system temperature sensors (multiple locations)
   HW API: Check cooling system status and fan speeds
   HW API: Monitor power supply voltages and current consumption
   HW API: Collect humidity sensors, vibration monitors
   HW API: Check safety interlock sensors and radiation LED status
   ```

7. **Storage and Memory Status Collection**
   ```
   HW API: Check system storage usage and health
   HW API: Monitor memory usage and system performance
   HW API: Read SD card and USB port status
   HW API: Check network connectivity and communication ports
   ```

8. **Hardware Status Compilation**
   ```
   HW API: Compile comprehensive hardware status report
   HW API -> MW API: Return system_health_data (device_statuses, sensor_readings, performance_metrics)
   ```

### Health Data Processing and Analysis

9. **Health Data Analysis**
   ```
   MW API: Process hardware status data
   MW API: Apply health status algorithms and thresholds
   MW API: Calculate system performance scores and trend analysis
   MW API: Identify critical issues, warnings, and maintenance needs
   MW API: Generate health status indicators (green/yellow/red)
   ```

10. **Historical Health Data Integration**
    ```
    MW API -> DB: getHistoricalHealthData(time_range) 
    DB -> MW API: Return historical_health_trends
    MW API: Combine current status with historical trends
    MW API: Calculate health trend directions and predictive indicators
    ```

11. **Health Dashboard Generation**
    ```
    MW API: Format health data for UI visualization
    MW API: Generate chart data for trends and performance metrics
    MW API: Create status summaries and alert messages
    MW API -> UI: Return health_dashboard_data (status_overview, detailed_metrics, charts, alerts)
    ```

### System Health Display

12. **Health Dashboard Rendering**
    ```
    UI: Receive comprehensive health dashboard data
    UI: Display system health overview with color-coded status indicators
    UI: Render real-time charts for key performance metrics
    UI: Show detailed component status with individual health scores
    UI: Display environmental sensor readings with trend indicators
    UI: Present alerts and maintenance recommendations
    ```

13. **Live Health Monitoring** (Optional continuous updates)
    ```
    MW API -> HW API: startContinuousHealthMonitoring() (async)
    HW API: Begin periodic health status updates (every 30 seconds)
    HW API -> MW API: healthStatusUpdate(updated_metrics) (async)
    MW API -> UI: updateHealthDisplay(real_time_data) (async)
    UI: Update charts and status indicators with live data
    ```

14. **Health Data Logging**
    ```
    MW API -> DB: logSystemHealthSnapshot(health_data, timestamp)
    MW API: Store health metrics for trend analysis and maintenance tracking
    ```

### API Sequence Diagram

```
User    UI       MW API            HW API    DB
 |       |         |                 |       |
 |------>|         |                 |       |  1. Navigate to System Health
 |       |-------->|                 |       |  2. GET /api/system/health-status
 |       | [loading indicator]       |       |  3. Display loading progress
 |       |         |---------------->|       |  4. pollAllDevices() (sync)
 |       |         |                 | [poll]|  5. Poll X-ray tube controller
 |       |         |                 | [poll]|  6. Poll DPP hardware
 |       |         |                 | [poll]|  7. Poll environmental sensors
 |       |         |                 | [poll]|  8. Poll storage/memory/network
 |       |         |<----------------|       |  9. Return system_health_data
 |       |         | [analyze data]  |       | 10. Process and analyze health data
 |       |         |------------------------->| 11. getHistoricalHealthData()
 |       |         |<-------------------------|12. Return historical_health_trends
 |       |         | [format dashboard]      | 13. Generate dashboard data
 |       |<--------|                 |       | 14. Return health_dashboard_data
 |       | [render dashboard]        |       | 15. Display health dashboard
 |       |         |---------------->|       | 16. startContinuousHealthMonitoring()
 |       |         |<----------------| [live]| 17. healthStatusUpdate() (periodic)
 |       |<--------| [live updates]  |       | 18. updateHealthDisplay()
 |       |         |------------------------->| 19. logSystemHealthSnapshot()
 |       |         |                 |       |
```

### System Health Components

**X-ray Tube Health Monitoring:**
- **Voltage/Current Status**: Real-time tube operating parameters
- **Temperature Monitoring**: Tube anode and cathode temperature tracking
- **Filament Health**: Filament current and aging indicators
- **Safety Status**: Interlock status and fault condition monitoring
- **Usage Metrics**: Operational hours and duty cycle tracking
- **Performance Indicators**: Beam stability and output consistency

**DPP and Detector Health:**
- **Detector Temperature**: Real-time detector temperature monitoring
- **High Voltage Status**: Detector bias voltage and stability
- **Count Rate Performance**: Spectrum processing efficiency and throughput
- **Resolution Metrics**: Energy resolution and peak shape quality
- **Communication Health**: Data communication integrity and timing
- **Calibration Status**: Detector calibration stability and drift

**Environmental System Health:**
- **System Temperature**: Multiple temperature sensors throughout system
- **Cooling System**: Fan speeds, cooling efficiency, thermal management
- **Power Supply Health**: Voltage levels, current consumption, power quality
- **Humidity Monitoring**: Environmental humidity sensors
- **Vibration Detection**: System stability and mechanical health
- **Safety Systems**: Radiation LED, interlocks, emergency stops

### Health Status Visualization

**Dashboard Layout:**
- **System Overview**: Color-coded overall system health indicator
- **Component Status Grid**: Individual component health with icons and status
- **Real-Time Charts**: Trend charts for key performance metrics
- **Environmental Panel**: Temperature, humidity, power, and cooling status
- **Alert Section**: Critical issues, warnings, and maintenance recommendations
- **Historical Trends**: Long-term health trend analysis and predictions

**Status Indicator System:**
- **Green (Healthy)**: All parameters within normal operating ranges
- **Yellow (Warning)**: Parameters approaching limits, maintenance recommended
- **Red (Critical)**: Parameters outside safe limits, immediate attention required
- **Gray (Unknown)**: Component not responding or data unavailable

### Health Monitoring Features

**Real-Time Monitoring:**
- **Live Updates**: Continuous monitoring with periodic status updates
- **Threshold Alerting**: Automatic alerts when parameters exceed thresholds
- **Trend Analysis**: Real-time trend calculation and display
- **Performance Scoring**: Overall system health score calculation

**Historical Analysis:**
- **Trend Charts**: Historical performance trends over time
- **Comparative Analysis**: Compare current status to historical baselines
- **Predictive Indicators**: Early warning indicators based on trends
- **Maintenance Scheduling**: Predictive maintenance recommendations

### Diagnostic Information

**Hardware Diagnostics:**
- **Component Versions**: Firmware and software version information
- **Communication Status**: Hardware communication health and timing
- **Error Logs**: Recent hardware errors and fault conditions
- **Performance Metrics**: Detailed performance statistics and benchmarks

**System Diagnostics:**
- **Storage Health**: Disk usage, read/write performance, error rates
- **Memory Usage**: RAM utilization and system resource consumption
- **Network Status**: Connectivity status and communication quality
- **Service Status**: System service health and operational status

### Maintenance Integration

**Maintenance Recommendations:**
- **Preventive Maintenance**: Scheduled maintenance recommendations
- **Component Replacement**: Aging component replacement suggestions
- **Calibration Reminders**: Calibration due dates and overdue notifications
- **Service Alerts**: Professional service requirement notifications

**Health History:**
- **Maintenance Log**: Record of all maintenance activities and their impact
- **Health Events**: Log of all health status changes and system events
- **Performance Tracking**: Long-term system performance trend analysis
- **Warranty Tracking**: Component warranty status and coverage information

### Key Design Decisions
- **Comprehensive Monitoring**: Monitor all critical system components and sensors
- **Real-Time Updates**: Provide live system health monitoring with periodic updates
- **Visual Dashboard**: Intuitive color-coded interface with charts and indicators
- **Historical Integration**: Combine current status with historical trend analysis
- **Predictive Analytics**: Early warning system for maintenance and failures
- **Maintenance Integration**: Link health monitoring to maintenance scheduling

### Success Criteria
- System health dashboard displays comprehensive status of all hardware components
- Real-time sensor data updates continuously with accurate readings
- Color-coded status indicators clearly communicate system health state
- Historical trend charts provide meaningful analysis of system performance
- Critical alerts and warnings notify users of issues requiring attention
- Maintenance recommendations help users optimize system performance
- Health monitoring data supports predictive maintenance scheduling
- Dashboard interface is intuitive and accessible to both users and technicians

---

## Use Case 9: Device Log Export

### Overview
Device log export system that enables users to export system logs, error reports, and diagnostic information to external storage or cloud services for troubleshooting, maintenance, and compliance purposes. System provides flexible log selection, multiple export destinations (USB, SD card, cloud), and secure transfer protocols while maintaining log integrity and comprehensive audit trails.

### Sequence Flow

```
Actor: User
UI Layer: User Interface
MW API: Middleware API  
HW API: Hardware API
DB: Database
Cloud Service: External Cloud Storage
```

**Step-by-Step API Flow:**

### Log Export Settings Access

1. **Settings Menu Navigation**
   ```
   User -> UI: Navigate to Settings menu
   UI: Display settings interface with System/Maintenance section
   UI: Show "Export Logs" button with last export status and timestamp
   ```

2. **Log Export Interface Display**
   ```
   User -> UI: Press "Export Logs" button
   UI -> MW API: GET /api/logs/export-status (async)
   MW API: Check available storage devices (USB, SD card)
   MW API: Check cloud service connectivity and authentication
   MW API -> UI: Return log_export_status (available_destinations, last_export_info)
   UI: Display log export interface with destination options
   ```

### Log Selection Process

3. **Available Logs Display**
   ```
   UI -> MW API: GET /api/logs/available-logs (date_range, log_types) (async)
   MW API -> DB: getAvailableLogs(date_filter, log_categories, user_permissions)
   DB -> MW API: Return log_list (log_files, dates, sizes, types, criticality)
   MW API -> UI: Return available_logs (log_summaries, total_size, categories)
   UI: Display selectable list of logs with checkboxes and categories
   UI: Show log details (date, type, size, criticality level, description)
   UI: Display total selected log size and estimated transfer time
   ```

4. **Log Category Selection**
   ```
   User -> UI: Selects desired log categories using checkboxes or filters
   UI: Show log category options (System, Hardware, Safety, Error, Audit, Performance)
   UI: Update selection summary (count, total size, estimated time)
   UI: Enable destination selection and export options
   ```

5. **Log Date Range and Filter Selection**
   ```
   User -> UI: Configures date range and severity filters
   UI: Display date picker and severity level filters (Info, Warning, Error, Critical)
   User -> UI: Selects export destination (USB, SD Card, or Cloud)
   UI: Display destination-specific options and available space
   ```

### USB Log Export Process

6. **USB Export Configuration** (If USB selected)
   ```
   User -> UI: Configures USB export settings (file format, compression)
   UI: Display USB device information and available space
   User -> UI: Confirms USB log export initiation
   UI -> MW API: POST /api/logs/export-usb (selected_logs, export_options) (async)
   ```

7. **USB Log Preparation and Transfer**
   ```
   MW API -> DB: retrieveLogData(selected_log_ids, include_system_info)
   DB -> MW API: Return complete_log_data (system_logs, hardware_logs, error_logs, audit_logs)
   MW API: Format logs according to user preferences (CSV, JSON, TXT, compressed)
   MW API: Package log files with system information and export metadata
   MW API -> HW API: initiateUSBLogTransfer(log_package, destination_path) (sync)
   HW API: Establish SFTP connection to USB device
   HW API: Create organized folder structure on USB destination
   HW API: Transfer log files using secure file transfer protocol
   HW API -> MW API: Return usb_log_transfer_progress (percent_complete, current_file)
   ```

### SD Card Log Export Process

8. **SD Card Export Configuration** (If SD Card selected)
   ```
   User -> UI: Configures SD card export settings (file organization, compression)
   UI: Display SD card information and available space
   User -> UI: Confirms SD card log export initiation
   UI -> MW API: POST /api/logs/export-sd (selected_logs, export_options) (async)
   ```

9. **SD Card Log Transfer**
   ```
   MW API -> DB: retrieveLogData(selected_log_ids, include_system_info)
   DB -> MW API: Return complete_log_data (system_logs, hardware_logs, error_logs, audit_logs)
   MW API: Format logs according to user preferences
   MW API -> HW API: initiateSDLogTransfer(log_package, folder_structure) (sync)
   HW API: Access SD card file system
   HW API: Create organized folder structure (by date, log type, severity)
   HW API: Copy log files to SD card destination folders
   HW API: Verify log file integrity after transfer
   HW API -> MW API: Return sd_log_transfer_progress (percent_complete, files_transferred)
   ```

### Cloud Log Export Process

10. **Cloud Export Configuration** (If Cloud selected)
    ```
    User -> UI: Selects cloud service and configures export settings
    UI: Display cloud authentication status and available storage
    User -> UI: Confirms cloud log export initiation
    UI -> MW API: POST /api/logs/export-cloud (selected_logs, cloud_service, export_options) (async)
    ```

11. **Cloud Log Upload**
    ```
    MW API -> DB: retrieveLogData(selected_log_ids, include_system_info)
    DB -> MW API: Return complete_log_data (system_logs, hardware_logs, error_logs, audit_logs)
    MW API: Format logs according to cloud service requirements
    MW API: Encrypt log data for secure cloud transmission
    MW API -> Cloud Service: uploadLogData(encrypted_log_package) (async)
    Cloud Service: Store logs with proper organization and metadata
    Cloud Service -> MW API: Return cloud_log_upload_progress (percent_complete, status)
    ```

### Progress Monitoring and Completion

12. **Real-Time Progress Updates**
    ```
    MW API -> UI: updateLogExportProgress(destination, percent_complete, current_operation) (async)
    UI: Display real-time progress bar with current file/operation
    UI: Show transfer speed, estimated time remaining, files completed
    UI: Update export status and allow user to cancel if needed
    ```

13. **Export Completion and Verification**
    ```
    MW API: Complete log transfer to selected destination
    MW API: Verify log integrity and transfer success
    MW API -> DB: updateLogExportStatus(log_ids, destination, export_timestamp)
    MW API -> UI: displayLogExportCompletion(export_results, verification_status) (async)
    UI: Show export completion message with transfer summary
    UI: Display successful transfers, any failures, total logs exported
    UI: Update export status indicators for completed logs
    ```

### API Sequence Diagram

```
User    UI       MW API    DB      HW API    Cloud Service
 |       |         |        |        |           |
 |------>|         |        |        |           |  1. Navigate to Export Logs
 |       |-------->|        |        |           |  2. GET /api/logs/export-status
 |       |<--------|        |        |           |  3. Return log_export_status
 |       |-------->|        |        |           |  4. GET /api/logs/available-logs
 |       |         |------->|        |           |  5. getAvailableLogs()
 |       |         |<-------|        |           |  6. Return log_list
 |       |<--------|        |        |           |  7. Return available_logs
 |       | [show log list]  |        |           |  8. Display selectable logs
 |------>|         |        |        |           |  9. Select logs & destination
 |       |-------->|        |        |           | 10. POST /api/logs/export-[destination]
 |       |         |------->|        |           | 11. retrieveLogData()
 |       |         |<-------|        |           | 12. Return complete_log_data
 |       |         |--------------->|           | 13. initiateUSBLogTransfer() [USB path]
 |       |         |                | [transfer]| 14. Transfer via SFTP protocol
 |       |         |<---------------|           | 15. Return log_transfer_progress
 |       |         |------------------------->| 16. uploadLogData() [Cloud path]
 |       |         |<-------------------------| 17. Return upload_progress
 |       |<--------|        |        |           | 18. updateLogExportProgress()
 |       | [show progress]  |        |           | 19. Display real-time progress
 |       |         |------->|        |           | 20. updateLogExportStatus()
 |       |<--------|        |        |           | 21. displayLogExportCompletion()
 |       | [show completion]|        |           | 22. Show export results
 |       |         |        |        |           |
```

### Log Categories and Types

**System Logs:**
- **Application Logs**: UI and MW API operation logs, user actions, system events
- **Database Logs**: Database operations, transactions, schema changes
- **Configuration Logs**: Settings changes, calibration updates, mode switches
- **Performance Logs**: System performance metrics, resource usage, timing data

**Hardware Logs:**
- **X-ray Tube Logs**: Tube operations, voltage/current events, temperature alerts
- **DPP Logs**: Detector operations, spectrum processing events, communication logs
- **Sensor Logs**: Environmental sensor readings, threshold alerts, sensor failures
- **Communication Logs**: Hardware communication events, protocol errors, timeouts

**Safety and Error Logs:**
- **Safety Event Logs**: Radiation LED events, interlock triggers, emergency stops
- **Error Logs**: System errors, hardware faults, communication failures
- **Warning Logs**: System warnings, maintenance alerts, threshold notifications
- **Audit Logs**: User authentication, data access, compliance events

### Export Format Options

**Text Formats:**
- **CSV Format**: Structured log data for spreadsheet analysis
- **JSON Format**: Structured log data for programmatic analysis
- **TXT Format**: Plain text log files for manual review
- **XML Format**: Structured format for system integration

**Compression Options:**
- **ZIP Compression**: Standard compression for reduced file size
- **TAR/GZIP**: Unix-style compression for technical users
- **Individual Files**: Uncompressed files for direct access
- **Encrypted Archives**: Password-protected compressed files

### Log Management Features

**Log Filtering:**
- **Date Range Selection**: Export logs from specific time periods
- **Severity Level Filtering**: Filter by Info, Warning, Error, Critical levels
- **Category Selection**: Choose specific log categories to export
- **Size Management**: Limit export size based on available storage

**Log Metadata:**
- **System Information**: Device serial number, firmware versions, configuration
- **Export Metadata**: Export timestamp, user information, selection criteria
- **Log Integrity**: Checksums and verification data for log files
- **Index Files**: Summary files describing exported log contents

### Security and Compliance

**Data Protection:**
- **Sensitive Data Filtering**: Remove or mask sensitive information from logs
- **Access Control**: User permission validation for log access and export
- **Encryption**: Encrypted transmission for cloud exports
- **Audit Trail**: Complete logging of all log export activities

**Compliance Features:**
- **Regulatory Compliance**: Log formats meeting regulatory requirements
- **Chain of Custody**: Maintain log integrity and custody information
- **Retention Policies**: Respect log retention and deletion policies
- **Privacy Protection**: Ensure user privacy in exported logs

### Troubleshooting and Diagnostic Support

**Diagnostic Packages:**
- **Complete Diagnostic Export**: All logs and system information for support
- **Error-Specific Exports**: Logs related to specific error conditions
- **Performance Analysis**: Logs focused on system performance issues
- **Maintenance Packages**: Logs relevant to maintenance and calibration

**Support Integration:**
- **Support Ticket Integration**: Link log exports to support requests
- **Remote Diagnostic**: Secure log sharing with technical support
- **Automated Analysis**: Basic log analysis and issue identification
- **Issue Correlation**: Cross-reference logs to identify related issues

### Key Design Decisions
- **Similar Architecture to Data Sync**: Leverage proven data sync patterns for log export
- **Comprehensive Log Categories**: Support all system log types for complete diagnostics
- **Flexible Filtering**: Allow users to select specific logs and date ranges
- **Multiple Export Formats**: Support various formats for different use cases
- **Secure Transfer**: Use same secure protocols as data sync (SFTP, encryption)
- **Diagnostic Focus**: Optimize for troubleshooting and maintenance workflows

### Success Criteria
- Users can easily select specific log categories and date ranges for export
- System successfully transfers logs to USB devices using secure SFTP protocol
- SD card export creates organized folder structure with complete log files
- Cloud export provides encrypted, secure log backup to external services
- Real-time progress feedback keeps users informed during transfer process
- Log integrity verification ensures all transfers are complete and accurate
- Export includes comprehensive system information for diagnostic purposes
- Multiple export formats support various analysis and troubleshooting needs

---