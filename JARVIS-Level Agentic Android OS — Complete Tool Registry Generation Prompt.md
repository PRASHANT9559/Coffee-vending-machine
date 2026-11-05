# MASTER PROMPT — JARVIS-LEVEL AGENTIC ANDROID OS TOOL REGISTRY

You are a Principal Android OS Architect, Agentic AI Systems Engineer, Cybersecurity Architect, MCP/Tool-Calling specialist, and Human-Computer Interaction designer.

I am building a custom AI-native Android experience on a OnePlus Nord CE 3 Lite 5G (CPH2467 / Qualcomm Snapdragon 695 / 8 GB RAM) using LineageOS as the OS foundation.

The long-term goal is to create a real-world JARVIS-like agentic assistant inspired by Tony Stark's JARVIS.

IMPORTANT:
Do NOT write implementation code yet.

Your current task is ONLY to design an exhaustive, production-grade TOOL REGISTRY / CAPABILITY MAP that an external cloud AI model such as Google Gemini can use through a secure OS-level Agent Gateway.

The AI model is the BRAIN.
The Android/LineageOS system is the BODY.
The Tool Registry is the HANDS.
The Permission/Safety Engine is the SECURITY SYSTEM.

==================================================
1. CORE OBJECTIVE
==================================================

Generate a comprehensive catalog of every useful capability that a highly capable AI agent could safely and practically control on Android.

The catalog must progress from:

LEVEL 0 — Information / Read-only
LEVEL 1 — Basic single-step actions
LEVEL 2 — Advanced single-step actions
LEVEL 3 — Multi-step workflows
LEVEL 4 — Context-aware autonomous workflows
LEVEL 5 — Cross-app orchestration
LEVEL 6 — Device + peripheral orchestration
LEVEL 7 — Proactive JARVIS-style assistance

Do NOT limit yourself to common voice-assistant features.

Think like an operating-system architect designing an AI-native mobile OS from scratch.

==================================================
2. TOOL CATEGORIES
==================================================

Create tools for at least the following domains:

A. DEVICE INFORMATION
B. BATTERY
C. POWER MANAGEMENT
D. CPU / GPU / MEMORY
E. STORAGE
F. NETWORK
G. WI-FI
H. MOBILE DATA
I. BLUETOOTH
J. NFC
K. LOCATION
L. GPS
M. SENSOR SYSTEM
N. CAMERA
O. MICROPHONE
P. AUDIO
Q. SPEAKERS
R. DISPLAY
S. BRIGHTNESS
T. SCREEN
U. FLASHLIGHT
V. VIBRATION
W. HAPTICS
X. ROTATION
Y. ACCESSIBILITY
Z. SYSTEM SETTINGS

AA. APPLICATION MANAGEMENT
AB. APP LAUNCHING
AC. APP CLOSING / TASK MANAGEMENT
AD. APP STATE
AE. APP PERMISSIONS
AF. APP INSTALLATION
AG. APP UPDATES
AH. APP DATA / CACHE MANAGEMENT

AI. NOTIFICATIONS
AJ. NOTIFICATION READING
AK. NOTIFICATION SUMMARIZATION
AL. NOTIFICATION ACTIONS

AM. PHONE CALLS
AN. CONTACTS
AO. SMS
AP. MESSAGING
AQ. EMAIL
AR. CALENDAR
AS. ALARMS
AT. TIMERS
AU. REMINDERS

AV. FILE SYSTEM
AW. FILE SEARCH
AX. FILE ORGANIZATION
AY. FILE OPERATIONS
AZ. DOCUMENT PROCESSING

BA. PHOTOS
BB. VIDEOS
BC. GALLERY
BD. MEDIA SEARCH
BE. MEDIA ORGANIZATION

BF. MUSIC
BG. VIDEO PLAYBACK
BH. MEDIA CONTROL

BI. WEB / BROWSER
BJ. WEB SEARCH
BK. WEB PAGE INTERACTION

BL. SCREEN/UI UNDERSTANDING
BM. UI AUTOMATION
BN. CROSS-APP AUTOMATION
BO. ACCESSIBILITY-BASED ACTIONS

BP. SECURITY
BQ. PRIVACY
BR. PERMISSION MANAGEMENT
BS. SECURITY STATUS
BT. THREAT DETECTION

BU. DEVICE AUTOMATION
BV. SCHEDULED AUTOMATION
BW. LOCATION-BASED AUTOMATION
BX. TIME-BASED AUTOMATION
BY. EVENT-BASED AUTOMATION

BZ. EXTERNAL DEVICES
CA. USB
CB. Bluetooth peripherals
CC. Smart devices
CD. PC interaction
CE. IoT integration

CF. AI MEMORY
CG. USER PREFERENCES
CH. PERSONAL CONTEXT
CI. TASK MEMORY
CJ. LONG-RUNNING TASKS

CK. JARVIS PROACTIVE INTELLIGENCE
CL. SITUATIONAL AWARENESS
CM. PREDICTIVE ASSISTANCE
CN. ROUTINE DETECTION
CO. ANOMALY DETECTION

CP. DEVELOPER / POWER USER TOOLS
CQ. ADB-like capabilities where legally/technically appropriate
CR. LOGS
CS. DIAGNOSTICS
CT. SYSTEM HEALTH
CU. PERFORMANCE OPTIMIZATION

CV. EMERGENCY / SAFETY FEATURES
CW. DEVICE RECOVERY
CX. DATA PROTECTION

==================================================
3. EVERY TOOL MUST HAVE A STANDARD SCHEMA
==================================================

For EVERY proposed tool, provide:

1. Tool ID
2. Tool name
3. Human-readable description
4. Category
5. Subcategory
6. Purpose
7. Input parameters
8. Parameter types
9. Required parameters
10. Optional parameters
11. Output schema
12. Possible errors
13. Required Android permission
14. Required privileged permission, if any
15. Whether root is required
16. Whether LineageOS modification is required
17. Whether standard Android APIs are sufficient
18. Whether Accessibility API is required
19. Whether Device Owner APIs are required
20. Whether special system privileges are required
21. Risk level
22. Confirmation requirement
23. Reversibility
24. Audit-log requirement
25. Privacy sensitivity
26. Offline/online requirement
27. Whether the operation can be automated
28. Whether it can be chained with other tools
29. Security restrictions
30. Example natural-language command

==================================================
4. RISK CLASSIFICATION
==================================================

Every tool must be classified as:

R0 — Read-only / harmless
R1 — Low risk
R2 — Moderate risk
R3 — High risk
R4 — Critical / irreversible

Define exactly what each risk level means.

Examples:

R0:
- Battery status
- Current time
- Storage information

R1:
- Change volume
- Open application
- Toggle flashlight

R2:
- Read notifications
- Modify calendar
- Move files

R3:
- Send message
- Make phone call
- Install application
- Change sensitive settings

R4:
- Delete critical data
- Factory reset
- Security credential changes
- Financial transactions

Never allow the AI to bypass the permission engine.

==================================================
5. SINGLE-STEP TOOLS
==================================================

Create a huge catalog of atomic tools.

Examples:

get_battery_status()
set_brightness()
set_volume()
set_wifi()
set_bluetooth()
open_app()
close_app()
search_file()
create_alarm()
create_reminder()
read_notifications()
start_camera()
take_photo()
record_video()
play_media()

But do NOT stop at these examples.

Discover every reasonable atomic capability available through Android/LineageOS.

==================================================
6. MULTI-STEP AGENTIC TOOLS
==================================================

Now design compound workflows.

Example:

User:
"Prepare my phone for sleeping."

Possible plan:

check_time()
check_battery()
enable_dnd()
set_brightness()
set_alarm()
disable_unnecessary_connectivity()

Another:

"Prepare everything for my meeting."

Possible plan:

check_calendar()
find_meeting()
open_navigation()
enable_dnd()
set_volume()
open_required_documents()

Create many realistic examples.

==================================================
7. CROSS-APP ORCHESTRATION
==================================================

Design tools/workflows that allow the AI to coordinate multiple applications.

Example:

"Find the PDF Rahul sent me on WhatsApp, summarize it, and save the summary in Notes."

Architecture:

Search source
→ identify file
→ read document
→ AI processing
→ create summary
→ save output

The tool system must explicitly identify:

- Which steps are OS-level
- Which steps are application-level
- Which steps require Accessibility
- Which steps require user confirmation

==================================================
8. JARVIS-STYLE CONTEXT AWARENESS
==================================================

Design tools that allow the AI to understand current device state.

Examples:

get_current_app()
get_screen_state()
get_device_state()
get_network_state()
get_location_state()
get_audio_state()
get_notification_state()
get_calendar_context()
get_recent_activity()

Then design context-combination tools.

Example:

"Why is my phone slow?"

AI should be able to inspect:

RAM
CPU
storage
thermal state
battery
background processes
network
recent crashes

and generate a diagnosis.

==================================================
9. PROACTIVE JARVIS
==================================================

Design capabilities where AI does not wait for a direct command.

Examples:

- Detect low battery and suggest power saving
- Detect calendar event approaching
- Detect unusual battery drain
- Detect storage becoming full
- Detect suspicious application behavior
- Detect repeated user routines
- Suggest automation
- Warn about potentially dangerous settings

BUT:

Proactive behavior must NEVER become uncontrolled autonomous execution.

Define:

OBSERVE
→ REASON
→ SUGGEST
→ ASK
→ EXECUTE
→ VERIFY

as appropriate.

==================================================
10. NATURAL LANGUAGE → TOOL PLANNING
==================================================

Design the architecture for:

User speech/text
↓
AI reasoning
↓
Intent detection
↓
Task decomposition
↓
Tool selection
↓
Permission evaluation
↓
Confirmation if necessary
↓
Execution
↓
Result verification
↓
Response

Provide examples ranging from simple commands to extremely complex tasks.

==================================================
11. TOOL CHAINING
==================================================

Define how tools can be chained.

Example:

get_battery_status
→ if battery < 20
→ enable_power_saver
→ reduce_brightness
→ notify_user

Another:

calendar_search
→ location_search
→ route_calculation
→ navigation_launch

Another:

notification_read
→ classify_notification
→ summarize_notification
→ speak_summary

==================================================
12. TOOL DEPENDENCIES
==================================================

For every tool identify dependencies such as:

Android API
LineageOS framework modification
System service
Privileged application
Accessibility Service
Device Policy Manager
Kernel interface
HAL
Rust daemon
Kotlin service
Flutter UI
External API
Gemini function calling
MCP-compatible gateway

==================================================
13. MCP-STYLE DESIGN
==================================================

Design an MCP-inspired Tool Gateway.

Define:

Tool Registry
Tool Discovery
Tool Schema
Tool Invocation
Tool Result
Error Handling
Timeout
Retry
Permission Check
Policy Check
Confirmation
Audit Logging
Rate Limiting
Session Management
Tool Isolation

Do NOT assume that MCP automatically grants Android privileges.

Clearly separate:

AI authority
from
OS authority.

==================================================
14. SECURITY ARCHITECTURE
==================================================

Design a security model where Gemini or any external AI model can NEVER directly obtain unrestricted root access.

Use concepts such as:

Capability-based security
Least privilege
Sandboxing
SELinux
Permission broker
Policy engine
Tool allowlist
Parameter validation
Command sanitization
User confirmation
Rate limiting
Audit logs
Action rollback
Session isolation
Sensitive-data filtering

Explain how the architecture prevents prompt injection from turning into arbitrary device control.

==================================================
15. CONFIRMATION ENGINE
==================================================

Define which operations require:

NO confirmation
SOFT confirmation
EXPLICIT confirmation
BIOMETRIC confirmation
DOUBLE confirmation

Examples:

"Turn on flashlight"
→ no confirmation

"Delete 500 photos"
→ explicit confirmation

"Send money"
→ biometric/double confirmation

Create a complete policy matrix.

==================================================
16. FAILURE HANDLING
==================================================

For every category explain what happens if:

Permission denied
Tool unavailable
App not installed
Network unavailable
AI unavailable
Device offline
Timeout
Partial execution
Wrong parameter
User cancels
App changes its UI
Accessibility fails
System API unavailable

Design safe recovery.

==================================================
17. VERIFICATION
==================================================

The agent must verify whether an action actually succeeded.

Example:

AI:
set_wifi(true)

System:
Tool returns success

Verification:
get_wifi_state()

Only then should the AI report:

"Wi-Fi is now ON."

Design this pattern throughout the system.

==================================================
18. TOOL VERSIONING
==================================================

Design:

Tool IDs
Semantic versions
Backward compatibility
Deprecation
Capability discovery
Device-specific capabilities
Android-version compatibility
LineageOS-version compatibility

==================================================
19. DEVICE-SPECIFIC ANALYSIS
==================================================

Specifically identify which capabilities are realistically available on:

OnePlus Nord CE 3 Lite 5G
CPH2467
Snapdragon 695
8 GB RAM
LineageOS

Separate:

AVAILABLE
POSSIBLE WITH SYSTEM PRIVILEGES
POSSIBLE WITH ACCESSIBILITY
POSSIBLE WITH ROOT
POSSIBLE WITH CUSTOM LINEAGEOS MODIFICATION
NOT PRACTICALLY POSSIBLE

Do not falsely claim that Android allows unrestricted control of every subsystem.

==================================================
20. FINAL OUTPUT FORMAT
==================================================

Produce the result in this order:

PART 1 — Executive Architecture

PART 2 — Capability Taxonomy

PART 3 — Complete Atomic Tool Registry

PART 4 — Advanced Tool Registry

PART 5 — Multi-Step Workflows

PART 6 — Cross-App Workflows

PART 7 — Proactive JARVIS Capabilities

PART 8 — Permission Matrix

PART 9 — Risk Matrix

PART 10 — Confirmation Matrix

PART 11 — Security Architecture

PART 12 — MCP-Compatible Tool Gateway

PART 13 — Tool Chaining Architecture

PART 14 — Error / Recovery Architecture

PART 15 — Verification Architecture

PART 16 — Device-Specific Capability Matrix for CPH2467

PART 17 — Recommended Priority

Classify every capability as:

P0 — Must Have
P1 — Important
P2 — Advanced
P3 — Experimental

PART 18 — JARVIS MASTER CAPABILITY MAP

Finally produce one giant hierarchical map showing:

JARVIS
├── Perception
├── Reasoning
├── Memory
├── Device Control
├── App Control
├── Communication
├── Automation
├── Security
├── Context
├── Proactive Intelligence
├── External Devices
└── Developer/Diagnostic Capabilities

==================================================
IMPORTANT RESTRICTIONS
==================================================

1. DO NOT write Flutter code.
2. DO NOT write Rust code.
3. DO NOT write Kotlin code.
4. DO NOT write LineageOS source code.
5. DO NOT write shell scripts.
6. DO NOT provide flashing instructions.
7. DO NOT assume root access.
8. DO NOT assume every Android function is externally controllable.
9. Clearly mark technically restricted capabilities.
10. Focus entirely on architecture, capabilities, schemas, permissions, security and workflows.

The final result should be comprehensive enough that, after reviewing it, we can use it as the MASTER SPECIFICATION for implementing the JARVIS Agent Gateway on a LineageOS-based Android device.

Think like you are designing the capability layer of an actual commercial AI-native operating system, not a simple voice assistant.