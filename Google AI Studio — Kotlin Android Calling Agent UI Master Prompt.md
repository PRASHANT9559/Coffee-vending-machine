You are a Senior Android Engineer, Kotlin Architect, UI/UX Designer, and Android System Integration Engineer.

Build the complete **Android Calling Agent mobile application UI** for an authorized enterprise call-center system.

IMPORTANT:
- The Android app is a dedicated **Calling Agent / Device Agent**.
- It will run on an Android phone containing a company-provided physical SIM.
- The app will later communicate with a Rust backend.
- Do NOT implement the Rust backend now.
- Do NOT implement the PostgreSQL database now.
- Do NOT implement real automated mass calling now.
- For now, build the complete Android application UI, navigation, local state architecture, permission handling, device-status monitoring UI, mock data, and integration interfaces/placeholders.
- Use mock/local data wherever backend data is not yet available.
- The system must only support authorized company-provided customer data and approved calling workflows.
- Do not implement random-number dialing, scraping, telecom restriction bypassing, CAPTCHA bypassing, or unauthorized calling.

==================================================
1. TECHNOLOGY
==================================================

Use:

- Kotlin
- Android Studio / Google AI Studio compatible Android project
- Modern Android architecture
- Jetpack Compose
- Material 3
- Kotlin Coroutines
- StateFlow / SharedFlow
- ViewModel
- Navigation Compose
- DataStore for local settings
- Room only if local persistence is genuinely required
- WorkManager for permitted background synchronization/maintenance tasks
- Android foreground service only where Android policy and the actual functionality require it
- Proper lifecycle-aware architecture
- Dependency Injection using a lightweight modern approach such as Hilt if justified

Use clean modular architecture.

Do not put the entire application into one giant Kotlin file.

Prefer files/modules of roughly 300–600 lines where practical.

==================================================
2. APPLICATION ROLE
==================================================

The phone application is NOT the main management dashboard.

It is a lightweight:

"Android Calling Agent"

Its responsibilities:

1. Register this Android device
2. Identify the assigned SIM/device
3. Connect to the Rust backend
4. Show connection status
5. Receive authorized call jobs
6. Display current customer/campaign information
7. Execute the permitted telephony workflow
8. Play the assigned TTS/audio
9. Receive permitted call-state events
10. Capture DTMF events where supported
11. Report call results
12. Maintain temporary local state during connectivity interruptions
13. Reconnect automatically
14. Report device health
15. Show errors and diagnostics
16. Allow authorized local device controls

The heavy management remains in the Flutter Control Center.

==================================================
3. DESIGN LANGUAGE
==================================================

Create a professional enterprise UI.

Design goals:

- Clean
- Modern
- Minimal
- Fast
- Easy to understand
- Large touch targets
- Excellent readability
- Suitable for a phone mounted on a desk
- Dark mode + light mode
- Material 3
- Responsive across common Android screen sizes
- Accessible
- Avoid unnecessary animations
- Show important status information immediately

Use a consistent design system:

- typography scale
- spacing system
- cards
- status chips
- buttons
- dialogs
- icons
- error states
- loading states
- empty states

Do not make it look like a generic social-media app.

It should look like professional industrial/enterprise device software.

==================================================
4. MAIN NAVIGATION
==================================================

Create these main screens:

1. Device Overview
2. Current Call
3. Call Queue
4. Campaign
5. Device/SIM Status
6. Connection
7. Activity / Call History
8. Diagnostics
9. Settings
10. Permissions
11. About / Device Information

Use bottom navigation only if it improves usability.

Otherwise use a simple navigation structure with the main dashboard as the home screen.

The primary screen must always make it obvious:

- Is the phone connected?
- Is the SIM ready?
- Is the device assigned?
- Is the agent running?
- Is there a current call?
- Is there an error?

==================================================
5. HOME / DEVICE OVERVIEW SCREEN
==================================================

Create a complete home screen.

Header:

"CALLING AGENT"

Show:

- Device name
- Device ID
- SIM name/ID
- Online/offline status

Example:

Device:
PHONE-001

SIM:
SIM-001

Status:
🟢 READY

Cards:

DEVICE STATUS
- Backend: Connected
- Internet: Connected
- SIM: Ready
- Telephony: Ready
- Agent Service: Running

DEVICE HEALTH
- Battery percentage
- Charging state
- Network type
- Signal strength where available
- Device temperature if safely available
- Storage availability
- App version

CAMPAIGN

- Campaign name
- Company name
- Campaign status
- Today's assigned calls
- Completed
- Remaining

CURRENT JOB

- Customer name
- Masked phone number
- Current status
- Call duration

Actions:

- Pause Agent
- Resume Agent
- Refresh Status
- Open Diagnostics

==================================================
6. CURRENT CALL SCREEN
==================================================

Create a dedicated current-call screen.

Show:

- Customer name
- Masked phone number
- Campaign name
- Offer / context
- Current call status
- Call timer
- Connection state

Example:

Customer:
Prashant

Number:
XXXXXX0010

Campaign:
Recharge Campaign

Status:
Calling

Duration:
00:24

Show the currently selected script/TTS information.

Example:

Voice:
Hindi Female 01

Script:

"नमस्ते {name} जी..."

Do not expose unnecessary sensitive information.

Controls must depend on the permitted workflow.

Possible states:

- Preparing
- Dialing
- Ringing
- Connected
- Playing TTS
- Waiting for response
- Processing response
- Completed
- Failed
- No Answer
- Busy
- Cancelled

==================================================
7. CALL RESULT SCREEN
==================================================

After a call, show result.

Possible result states:

- Interested
- Callback
- Not Interested
- No Answer
- Busy
- Failed
- Unknown

Show:

Customer
Call duration
SIM
Campaign
Result
Timestamp

Then:

"Waiting for next job..."

Do not automatically retry a customer locally.

Retry policy must ultimately be controlled by the backend/campaign rules.

==================================================
8. DTMF SCREEN / COMPONENT
==================================================

Create a UI component for DTMF results.

Example:

Customer Response

1 — Interested
2 — Callback
3 — Not Interested

Show the detected option.

Example:

Detected:
2 — Callback

This is a display/reporting component.

Do not invent unsupported telephony capabilities.

==================================================
9. CALL QUEUE SCREEN
==================================================

Show the jobs received from the backend.

Example:

QUEUE

#001  Prashant     Waiting
#002  Rahul        Waiting
#003  Amit         Waiting
#004  Neha         Waiting

Show:

- Queue count
- Current job
- Waiting jobs
- Failed synchronization jobs

The phone should NOT independently generate an unlimited calling list.

Only backend-authorized jobs should appear here.

==================================================
10. CAMPAIGN SCREEN
==================================================

Show:

Company
Campaign
Campaign ID
Campaign status

Statistics:

Assigned
Completed
Remaining
Failed
Callbacks

Show:

Current script
Selected TTS voice
Current calling configuration

The phone app should mostly display configuration received from backend.

Do not create company-level campaign management features here.

==================================================
11. SIM / TELEPHONY STATUS SCREEN
==================================================

This screen is extremely important.

Show:

SIM Status
- SIM present / absent
- SIM slot
- Network operator where Android legally exposes it
- Network registration state where available
- Signal information where available
- Data connection
- Telephony availability

Example:

SIM-001
Slot 1
🟢 Ready

Network:
4G

Signal:
Good

Also show safe diagnostic information.

Do not attempt to bypass carrier restrictions or obtain unavailable information.

==================================================
12. CONNECTION SCREEN
==================================================

Create a complete connection diagnostics screen.

Show:

INTERNET
🟢 Connected

BACKEND
🟢 Connected

WEBSOCKET
🟢 Connected

LAST SYNC
21:45:12

SERVER LATENCY
42 ms

AUTHENTICATION
🟢 Valid

DEVICE REGISTRATION
🟢 Approved

Show:

Reconnect

Refresh

Diagnostics

If disconnected:

Internet ❌
Backend ❌
WebSocket ❌

Show clear explanations.

==================================================
13. BACKGROUND AGENT STATUS
==================================================

Create a screen explaining the agent/background state.

Show:

Agent Service:
🟢 Running

Background execution:
Allowed / Restricted

Battery optimization:
Optimized / Exempt / Unknown

Notifications:
Allowed / Disabled

Required service state:
Ready / Needs attention

If the app requires a foreground service for an actual permitted function, provide the correct Android UI to guide the user through the required system settings.

Do NOT attempt to silently bypass Android background execution restrictions.

==================================================
14. PERMISSIONS SCREEN
==================================================

Create a dedicated permission manager.

Detect and display only permissions genuinely required by implemented functionality.

Potential categories include, depending on the Android API and actual implementation:

- Phone/telephony-related permissions
- Notifications
- Network state
- Foreground service requirements
- Bluetooth only if actually required
- Microphone only if actual STT/recording functionality requires it
- Storage/media only if actually required

For every permission show:

Permission name
Why it is needed
Current state

Example:

Notifications
Required for agent status notifications

Status:
🟢 Granted

Buttons:

Grant
Open Settings
Retry Check

Never request unnecessary permissions.

Never request permissions just because they might be useful later.

==================================================
15. SETTINGS SCREEN
==================================================

Create:

GENERAL

- Device name
- Theme
- Language
- Auto-start behavior if permitted
- Notification preferences

CONNECTION

- Backend status
- Server URL placeholder
- WebSocket status
- Reconnect behavior

CALLING AGENT

- Agent enabled
- Local diagnostic mode
- Call-job synchronization

AUDIO / TTS

- Selected voice
- Volume
- Audio test
- TTS provider/profile received from backend

DIAGNOSTICS

- Logs
- Clear temporary cache
- Run diagnostic test

SECURITY

- Device registration
- Authentication status
- Session status

Do not store secrets in plain text.

==================================================
16. DEVICE REGISTRATION SCREEN
==================================================

First-launch flow:

1. Welcome
2. Device information
3. Device ID
4. App version
5. Network status
6. SIM availability
7. Permission check
8. Backend connection
9. Registration request
10. Waiting for admin approval
11. Registered successfully

Example:

DEVICE REGISTRATION

Device:
PHONE-001

Status:
Waiting for approval

Message:

"This device must be approved by the administrator before it can receive calling jobs."

Do not allow unauthorized devices to receive jobs.

==================================================
17. DIAGNOSTICS SCREEN
==================================================

Create a detailed diagnostics page.

Sections:

DEVICE

- Android version
- Manufacturer
- Model
- App version
- Device ID

NETWORK

- Connection type
- Network availability
- Signal information where available

SIM

- SIM state
- Slot
- Operator information where legally available

BACKEND

- API connection
- WebSocket connection
- Last successful synchronization
- Latency

AGENT

- Service state
- Current job
- Queue state

BATTERY

- Battery percentage
- Charging state

STORAGE

- Available storage

AUDIO

- Audio output state
- TTS test state

Provide:

[Run Full Diagnostic]

Return clear pass/warning/fail states.

==================================================
18. ACTIVITY / CALL HISTORY
==================================================

Create a local activity screen.

Show recent events:

10:32
Call job received

10:33
Call started

10:34
TTS playback completed

10:34
DTMF received: 1

10:34
Result submitted

10:35
Next job received

Allow filtering:

- All
- Calls
- Connection
- Errors
- Synchronization

Do not store unnecessary customer-sensitive data locally.

==================================================
19. ERROR HANDLING UI
==================================================

Create proper UI for:

- SIM missing
- SIM unavailable
- No network
- Backend unavailable
- Authentication expired
- Device not approved
- WebSocket disconnected
- Call job expired
- TTS failure
- Audio failure
- Telephony failure
- Permission missing
- Battery restriction
- App service stopped

Each error must show:

What happened
Why it matters
What the user can do

Avoid technical jargon where unnecessary.

==================================================
20. OFFLINE MODE
==================================================

The app must handle temporary connectivity loss gracefully.

When offline:

Show:

OFFLINE MODE

Internet:
❌

Backend:
❌

Agent:
Paused / Waiting

Do not invent new calling jobs while offline.

Preserve only the minimum safe local state required for recovery.

When connection returns:

1. Authenticate
2. Verify device
3. Synchronize state
4. Report previous acknowledged events
5. Resume only if backend authorizes it

==================================================
21. ANDROID SYSTEM INTEGRATION
==================================================

Implement clean interfaces/classes for future integration with:

- Android Telephony APIs
- Telecom APIs where appropriate
- Call state observation
- Audio/TTS
- Network state
- Battery state
- Connectivity
- Device registration
- Foreground service where genuinely required
- Notifications
- WorkManager
- DataStore

Do not claim that an API can perform something that Android does not permit.

Where functionality depends on Android version, create an abstraction:

interface TelephonyController

interface DeviceHealthMonitor

interface NetworkMonitor

interface TtsController

interface BackendConnection

interface CallJobHandler

interface PermissionManager

Use mock implementations for the UI prototype.

==================================================
22. BACKEND INTERFACE PLACEHOLDERS
==================================================

Create API/data models for future Rust backend integration.

Example concepts:

DeviceRegistration
DeviceStatus
SimStatus
CampaignInfo
CallJob
CallState
CallResult
TtsProfile
ScriptTemplate
CallbackJob
BackendHealth

Use clear DTO/domain separation.

Do not implement the Rust backend.

Use fake repository implementations so the UI works immediately.

==================================================
23. SECURITY
==================================================

Follow secure Android practices.

Do NOT:

- hard-code API keys
- hard-code passwords
- store credentials in source code
- expose sensitive customer data in logs
- expose full phone numbers unnecessarily
- store sensitive data unnecessarily
- bypass Android security
- use accessibility services as a workaround for prohibited automation
- bypass telecom restrictions

Use secure storage mechanisms where credentials/tokens genuinely need local persistence.

==================================================
24. NOTIFICATION SYSTEM
==================================================

Create notification channels for relevant events such as:

- Agent offline
- Backend disconnected
- Device approval
- Diagnostic warning
- Campaign paused
- Device error

Do not spam notifications.

Notifications must respect Android requirements.

==================================================
25. MOCK DATA MODE
==================================================

The application MUST work without backend connectivity during development.

Create:

MockDeviceRepository
MockCallRepository
MockCampaignRepository
MockTtsRepository

Use realistic sample data:

Device:
PHONE-001

SIM:
SIM-001

Campaign:
Recharge Campaign

Customer:
Prashant

Status:
Calling

Do not use real customer information.

==================================================
26. UI STATES
==================================================

Every important screen must support:

- Loading
- Success
- Empty
- Error
- Offline
- Unauthorized
- Disabled
- Reconnecting

Do not design only the happy path.

==================================================
27. PROJECT STRUCTURE
==================================================

Use a clean structure similar to:

app/
  src/main/
    java/.../
      core/
        network/
        security/
        permissions/
        logging/

      data/
        local/
        remote/
        repository/

      domain/
        model/
        repository/
        usecase/

      feature/
        dashboard/
        currentcall/
        queue/
        campaign/
        simstatus/
        connection/
        activity/
        diagnostics/
        permissions/
        settings/
        registration/

      service/
        calling/
        connectivity/
        sync/

      ui/
        components/
        theme/
        navigation/

Do not create huge monolithic files.

==================================================
28. UI QUALITY REQUIREMENTS
==================================================

The result must look like a real production Android application.

Do not generate:

- placeholder rectangles everywhere
- empty screens
- fake navigation with no content
- unfinished buttons
- TODO screens
- broken previews
- giant single-file code
- random colors
- inconsistent spacing

Every visible button must either work in mock mode or clearly indicate why it is disabled.

==================================================
29. FINAL DELIVERABLE
==================================================

Generate the complete Kotlin Android UI project.

Include:

1. All screens
2. Navigation
3. Material 3 theme
4. Dark/light mode
5. ViewModels
6. UI state models
7. Mock repositories
8. Device registration flow
9. Permission manager UI
10. Device health UI
11. SIM status UI
12. Connection diagnostics
13. Current call UI
14. Call queue UI
15. Campaign UI
16. Activity history
17. Settings
18. Error states
19. Offline states
20. Notification setup
21. Background-agent status UI
22. Interfaces for future Rust backend
23. Interfaces for future telephony integration
24. Interfaces for future TTS/STT integration
25. Proper AndroidManifest configuration for only the permissions actually required by the implemented prototype
26. README with setup instructions

==================================================
30. IMPORTANT IMPLEMENTATION RULE
==================================================

Do not stop after creating only a visual mockup.

Build a functional UI prototype with:

- real navigation
- working state changes
- mock device status
- mock SIM status
- mock call queue
- mock current call
- mock campaign
- mock diagnostics
- permission state detection where available
- network state detection
- battery state detection
- connection state simulation
- error-state simulation

However, do NOT connect to a real telephony service or perform automated mass calling in this UI prototype.

The final project must be structured so that the Rust backend and authorized Android telephony layer can be integrated later without rewriting the UI architecture.

Before finishing, perform a complete compile-oriented review:

- imports
- Gradle dependencies
- AndroidManifest
- Compose APIs
- navigation
- state management
- lifecycle handling
- permissions
- Android API compatibility
- null safety
- coroutine scopes
- background-service rules
- build configuration

Fix all obvious compile errors and incomplete references.

The goal is a polished, production-quality **Android Calling Agent UI foundation**, not a simple demo screen.