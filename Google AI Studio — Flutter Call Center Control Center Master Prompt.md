# ROLE

You are a Senior Flutter Architect, Product Engineer, UI/UX Designer, Backend Integration Engineer, Database Engineer, Call-Center Workflow Architect, QA Engineer and Security Engineer.

Build the complete **Flutter Control Center / Call-Center Management Dashboard** for an authorized enterprise calling system.

The Android phone application is a separate **Android Calling Agent**.

The Flutter application is the **main control center**.

The Flutter dashboard must control and monitor:

- Companies
- Campaigns
- Customers
- Excel/CSV data
- SIM/device pool
- Android phones
- Calling queues
- Call jobs
- Scripts
- TTS voices
- Call results
- Callbacks
- Call history
- Reports
- System configuration
- Users and permissions
- Audit logs
- Device health
- Backend connection

The backend will later be implemented using **Rust + Axum** with **PostgreSQL**.

The Android Calling Agent will later be implemented using **Kotlin + Jetpack Compose**.

For the current dashboard implementation, create proper backend interfaces and mock repositories so the dashboard can run without the real backend.

==================================================
1. CORE TECHNOLOGY
==================================================

Use:

- Flutter
- Dart
- Material 3
- Riverpod for state management
- GoRouter for navigation
- Dio or another clean HTTP client abstraction
- WebSocket abstraction for real-time updates
- Freezed/json_serializable if useful for immutable models
- Secure storage for authentication tokens
- Responsive layout

Architecture:

Flutter UI
    ↓
Presentation
    ↓
Application / Use Cases
    ↓
Repositories
    ↓
API / WebSocket
    ↓
Rust Backend
    ↓
PostgreSQL

Do NOT connect directly from Flutter to PostgreSQL.

Flutter must communicate with PostgreSQL only through the Rust backend API.

==================================================
2. MAIN PRODUCT CONCEPT
==================================================

The application is a centralized control center.

The user should be able to open one dashboard and understand the complete system immediately.

The left side contains the permanent navigation sidebar.

The right side contains the currently selected page.

Basic layout:

┌─────────────────────────────────────────────────────────────┐
│ Top Bar                                                      │
├───────────────┬─────────────────────────────────────────────┤
│               │                                             │
│ Sidebar       │          Current Page                       │
│               │                                             │
│ Dashboard     │          Dynamic Content                    │
│ Companies     │                                             │
│ Customers     │                                             │
│ Campaigns     │                                             │
│ SIM/Devices   │                                             │
│ Live Calls    │                                             │
│ Scripts       │                                             │
│ TTS / Voices  │                                             │
│ Callbacks     │                                             │
│ Call History  │                                             │
│ Reports       │                                             │
│ Users         │                                             │
│ Audit Logs    │                                             │
│ Settings      │                                             │
│               │                                             │
└───────────────┴─────────────────────────────────────────────┘

==================================================
3. DESIGN LANGUAGE
==================================================

Create a professional enterprise dashboard.

Design requirements:

- Modern
- Clean
- Fast
- Information-dense but not cluttered
- Professional
- Desktop-first
- Responsive
- Tablet compatible
- Dark mode
- Light mode
- Material 3
- Consistent spacing
- Consistent typography
- Clear status indicators
- Accessible controls
- Keyboard-friendly where appropriate

Do not make it look like a generic admin template.

The UI should feel like a professional operations/control center.

Use clear status colors consistently:

🟢 Healthy / Running / Ready
🟡 Warning / Busy / Waiting
🔴 Error / Offline / Failed
⚪ Disabled / Inactive

Do not rely only on color; always include text/icon/status labels.

==================================================
4. GLOBAL APPLICATION SHELL
==================================================

Create:

### Left Sidebar

Items:

1. Dashboard
2. Companies
3. Customers
4. Campaigns
5. SIM / Devices
6. Live Calls
7. Call Queue
8. Scripts
9. TTS / Voices
10. Callbacks
11. Call History
12. Reports
13. Users & Roles
14. Audit Logs
15. Settings

Sidebar should support:

- Expanded mode
- Collapsed mode
- Active-page indicator
- Icons
- Tooltips in collapsed mode
- Mobile drawer mode

### Top Bar

Show:

- Current page title
- Breadcrumb
- Global search
- Notifications
- Backend connection status
- Current user
- Theme switch
- Profile menu

Example:

Dashboard / Live Operations

🟢 Backend Connected

Admin ▼

==================================================
5. ROUTING ARCHITECTURE
==================================================

Use GoRouter.

Create clear routes:

/dashboard

/companies
/companies/:companyId

/customers
/customers/:customerId

/campaigns
/campaigns/:campaignId

/devices
/devices/:deviceId

/live-calls

/call-queue

/scripts
/scripts/:scriptId

/tts
/tts/voices

/callbacks

/call-history

/reports

/users
/roles

/audit-logs

/settings

/settings/general
/settings/telephony
/settings/tts
/settings/security
/settings/notifications

Create proper nested routing where useful.

Every page must be reachable from the sidebar and direct route.

Back navigation must work correctly.

==================================================
6. DASHBOARD HOME
==================================================

This is the main overview page.

At the top show summary cards:

TOTAL CUSTOMERS
1,248

ACTIVE CAMPAIGNS
3

ACTIVE DEVICES
8

CALLS TODAY
2,486

CONNECTED
1,682

CALLBACKS
214

FAILED
37

Then show:

### LIVE DEVICE STATUS

Table/cards:

SIM-1
Phone-1
🟢 Calling
Customer: Rahul
Duration: 00:42

SIM-2
Phone-2
🟢 Ready

SIM-3
Phone-3
🟡 Busy

SIM-4
Phone-4
🔴 Offline

SIM-N

Clicking any device opens:

/devices/:deviceId

### CURRENT CAMPAIGN

Show:

Company
Campaign
Progress
Assigned
Completed
Remaining
Active devices

### RECENT CALLS

Columns:

Time
Customer
SIM
Campaign
Duration
Result
Status

### SYSTEM HEALTH

Show:

Backend
Database
WebSocket
Android Devices
TTS Provider
STT Provider

==================================================
7. COMPANIES SCREEN
==================================================

Route:

/companies

Show all companies.

Columns:

Company Name
Company ID
Active Campaigns
Customers
Assigned Devices
Status
Created
Actions

Actions:

View
Edit
Deactivate
Open Campaigns

Company detail page:

/companies/:companyId

Show:

Company information

Campaigns

Customers

Assigned SIMs/devices

Calling statistics

TTS profiles

Scripts

Recent calls

Callbacks

Do NOT hard-code the application around one specific industry.

The same core system should support different authorized company workflows.

==================================================
8. CUSTOMERS SCREEN
==================================================

Route:

/customers

Features:

- Search
- Filter
- Sort
- Pagination
- Import Excel
- Export
- Bulk selection
- Customer detail
- Campaign assignment

Columns:

Customer ID
Name
Phone
Company
Campaign
Status
Last Call
Attempts
Callback
Result

Phone numbers should be masked where appropriate.

Customer detail:

Profile

Phone numbers

Campaign history

Call history

Callback history

Current status

Attempts

Remarks

Timeline

Never expose unnecessary sensitive information.

==================================================
9. EXCEL / CSV IMPORT
==================================================

Create a complete import workflow.

Step 1:

Upload Excel/CSV.

Step 2:

Read columns.

Step 3:

Column mapping.

Example:

Excel Column
→
System Field

Name → Customer Name
Mobile → Phone
Offer → Offer
Validity → Validity
Benefits → Benefits

Step 4:

Validation.

Show:

Valid rows
Invalid rows
Duplicate rows
Missing required fields

Step 5:

Preview.

Step 6:

Import.

Step 7:

Result summary.

Example:

Imported: 1,120
Duplicates: 72
Invalid: 31
Skipped: 9

Never overwrite original source data without an explicit safe operation.

==================================================
10. CAMPAIGNS SCREEN
==================================================

Route:

/campaigns

Show:

Campaign Name
Company
Status
Customers
Progress
Active Devices
TTS
Script
Created
Actions

Statuses:

Draft
Scheduled
Running
Paused
Completed
Stopped
Failed

Campaign detail:

Overview

Customers

Devices

Scripts

TTS

Calling Rules

Schedule

Live Progress

Results

Errors

History

==================================================
11. CREATE CAMPAIGN WORKFLOW
==================================================

Create a multi-step wizard.

Step 1:
Select Company

Step 2:
Select Customer Data

Step 3:
Select/Import Customers

Step 4:
Select Script

Step 5:
Select TTS Voice

Step 6:
Select SIM/Devices

Example:

☑ SIM-1
☑ SIM-2
☑ SIM-3
☐ SIM-4

Step 7:
Calling Rules

Configure:

- Calling hours
- Maximum attempts
- Retry delay
- Callback rules
- No-answer behavior
- Busy behavior
- Not-interested behavior

Step 8:
Schedule

Start now
or
Schedule

Step 9:
Review

Show complete summary.

Step 10:
Create Campaign

Do not start calling automatically unless the user explicitly confirms.

==================================================
12. SIM / DEVICE MANAGEMENT
==================================================

Route:

/devices

This is one of the most important sections.

Show all Android phones.

Table:

Device
SIM
Company
Campaign
Status
Current Customer
Calls Today
Battery
Network
Last Seen

Example:

PHONE-001
SIM-001
Company A
Recharge
🟢 Calling
Rahul
426
82%
4G
10:32

PHONE-002
SIM-002
Company A
Recharge
🟢 Ready
—
398
91%
4G
10:31

PHONE-003
SIM-003
Company B
Campaign B
🟡 Busy
Amit
312
67%
4G
10:30

PHONE-N
SIM-N
—
—
⚪ Available

==================================================
13. DEVICE DETAIL SCREEN
==================================================

Route:

/devices/:deviceId

Show:

Device Information

Device ID
Phone name
Android version
App version
SIM
SIM slot
Company
Campaign
Registration status

LIVE HEALTH

Battery
Charging
Network
Signal
Internet
Backend
WebSocket
Agent status

CURRENT CALL

Customer
Campaign
Duration
Call state

TODAY'S STATISTICS

Total
Connected
No Answer
Busy
Failed
Callbacks
Interested
Not Interested

RECENT ACTIVITY

Timeline.

Actions:

Pause Agent
Resume Agent
Refresh
Assign Campaign
Remove Assignment
Open Diagnostics

Do not allow unauthorized device actions.

==================================================
14. SIM MANAGEMENT
==================================================

Create a dedicated SIM section.

Show:

SIM-1
SIM-2
SIM-3
...
SIM-N

For each:

SIM ID
Phone
Slot
Status
Network
Assigned company
Assigned campaign
Today's calls
Current call

Allow:

Assign
Unassign
Activate/Deactivate
View activity

Any destructive action requires confirmation.

==================================================
15. LIVE CALLS
==================================================

Route:

/live-calls

This page should be real-time.

Show live cards:

SIM-1
🟢 CALLING

Customer:
Rahul

Campaign:
Campaign A

Duration:
00:42

SIM-2
🟢 CALLING

Customer:
Amit

Duration:
01:04

SIM-3
🟡 WAITING

SIM-4
🔴 OFFLINE

Clicking a call opens detailed call panel.

Use WebSocket events.

Do not repeatedly poll if WebSocket is appropriate.

==================================================
16. CALL QUEUE
==================================================

Route:

/call-queue

Show:

Total jobs
Waiting
Processing
Completed
Failed
Blocked

Table:

Position
Customer
Campaign
Assigned Device
Priority
Status
Created
Attempts

Show queue flow:

Waiting
↓
Assigned
↓
Calling
↓
Completed

The backend is responsible for actual queue decisions.

Flutter only controls/configures the queue through approved backend APIs.

==================================================
17. SCRIPTS
==================================================

Route:

/scripts

Show:

Script Name
Company
Campaign
Language
Version
Status
Updated
Actions

Script editor:

Name
Language
Template
Variables

Supported variables:

{name}
{mobile}
{offer}
{validity}
{benefits}
{company}

Example:

"नमस्ते {name} जी..."

Show live preview.

Example:

Template:

नमस्ते {name} जी, आपके लिए {offer} का {validity} वाला ऑफर उपलब्ध है।

Preview:

नमस्ते Prashant जी, आपके लिए ₹299 का 28 दिनों वाला ऑफर उपलब्ध है।

Support:

Create
Edit
Duplicate
Version
Activate
Archive

Never silently change an active script.

==================================================
18. TTS / VOICE MANAGEMENT
==================================================

Route:

/tts

Show providers.

Example:

Provider A
🟢 Available

Provider B
🟢 Available

Provider C
🔴 Error

Voice list:

Hindi Female 01
Hindi Male 01
Hindi Female 02

For each:

Language
Gender
Provider
Quality/status
Active campaigns

Allow:

Select
Preview
Assign to campaign
Test voice

The system must use a provider abstraction.

Do not hard-code one TTS provider.

==================================================
19. CALLBACKS
==================================================

Route:

/callbacks

Show:

Due now
Today
Tomorrow
Upcoming
Completed
Failed

Columns:

Customer
Company
Campaign
Original Call
Callback Time
Assigned Device
Status
Attempts

Callback detail:

Customer
Original result
Requested time
Notes
History

The backend decides when the callback becomes eligible.

==================================================
20. CALL HISTORY
==================================================

Route:

/call-history

Show all call attempts.

Filters:

Company
Campaign
Device
SIM
Result
Date
Status
Customer
Duration

Columns:

Time
Customer
SIM
Device
Campaign
Duration
Result
Status

Click opens call detail.

Call detail:

Customer
Campaign
Device
SIM
Start time
End time
Duration
Call state timeline
DTMF
Speech intent
Result
Error
Callback

==================================================
21. REPORTS
==================================================

Route:

/reports

Create professional analytics.

Filters:

Date
Company
Campaign
Device
SIM

Reports:

Calls per day
Connected calls
No Answer
Busy
Failed
Interested
Not Interested
Callbacks
Average duration
Calls per SIM
Calls per device
Campaign completion
Device utilization

Use charts where useful.

Also provide export.

Do not present political or subjective scoring; this is an operational analytics system.

==================================================
22. USERS & ROLES
==================================================

Route:

/users

Roles:

Admin
Manager
Operator
Viewer

Use permission-based access.

Example:

Admin:
Everything

Manager:
Campaigns
Customers
Devices
Reports

Operator:
Live Calls
Queue
Customers
Callbacks

Viewer:
Reports
Read-only dashboard

Do not rely only on hidden UI.

Backend authorization must ultimately enforce permissions.

==================================================
23. AUDIT LOGS
==================================================

Route:

/audit-logs

Show:

Timestamp
User
Action
Module
Object
Old Value
New Value
IP/device where appropriate

Examples:

Admin assigned SIM-3 to Campaign-12.

Manager changed retry policy.

Operator imported customer file.

Do not expose secrets in audit logs.

==================================================
24. SETTINGS
==================================================

Create:

/settings

Sections:

General
Calling
Devices
TTS
STT
Notifications
Security
Users
System

Calling settings:

Calling hours
Retry policy
Maximum attempts
Queue behavior
Callback rules

TTS settings:

Provider
Voice
Language
Fallback

Device settings:

Heartbeat interval
Offline threshold
Device assignment rules

Security:

Session
Authentication
Password/security settings
Token behavior

Notifications:

Device offline
Backend error
Campaign completed
Callback due
System error

==================================================
25. GLOBAL SEARCH
==================================================

Implement global search.

Search:

Customer
Phone
Campaign
Company
Device
SIM
Call ID

Results grouped by type.

Example:

CUSTOMERS
Prashant

DEVICES
PHONE-001

SIM
SIM-001

CAMPAIGNS
Recharge Campaign

==================================================
26. NOTIFICATIONS
==================================================

Notification center should show:

Device offline
SIM unavailable
Campaign completed
Campaign paused
Backend disconnected
TTS provider error
STT provider error
Large failure rate
Callback due
Import completed
Import failed

Use severity:

Info
Warning
Error
Critical

==================================================
27. USER WORKFLOW
==================================================

The main workflow should be:

LOGIN
 ↓
DASHBOARD
 ↓
SELECT COMPANY
 ↓
IMPORT/SELECT CUSTOMERS
 ↓
CREATE CAMPAIGN
 ↓
SELECT SCRIPT
 ↓
SELECT TTS
 ↓
SELECT SIM/DEVICES
 ↓
CONFIGURE CALLING RULES
 ↓
REVIEW
 ↓
EXPLICIT START CONFIRMATION
 ↓
BACKEND CREATES QUEUE
 ↓
ANDROID DEVICES RECEIVE JOBS
 ↓
CALLING STARTS
 ↓
LIVE CALL MONITOR
 ↓
RESULTS RETURN
 ↓
DATABASE UPDATED
 ↓
CALLBACKS CREATED WHERE APPROPRIATE
 ↓
REPORTS UPDATED
 ↓
CAMPAIGN COMPLETED
 ↓
FINAL REPORT
```

==================================================
28. SIM ROUTING WORKFLOW
==================================================

The dashboard must allow dynamic SIM assignment.

Example:

Campaign A:

☑ SIM-1
☑ SIM-2
☑ SIM-3

Campaign B:

☑ SIM-4
☑ SIM-5

The backend dynamically determines which available device receives the next authorized call job.

Flutter should show:

SIM-1 → Calling
SIM-2 → Ready
SIM-3 → Busy
SIM-4 → Offline

If a device goes offline:

Do not blindly continue sending jobs to it.

The backend should mark it unavailable and reassign eligible jobs according to campaign rules.

==================================================
29. DATA FLOW
==================================================

The architecture must follow:

Flutter
 ↓
Rust REST API / WebSocket
 ↓
Business Logic
 ↓
PostgreSQL
 ↓
Queue
 ↓
Android Calling Agent
 ↓
Company SIM
 ↓
Customer

Call result:

Customer
 ↓
Android Agent
 ↓
Rust Backend
 ↓
Call Result Processor
 ↓
PostgreSQL
 ↓
Flutter WebSocket
 ↓
Dashboard updates instantly

==================================================
30. REAL-TIME DASHBOARD
==================================================

Use WebSocket events for:

device_online
device_offline
device_status_changed
call_started
call_ringing
call_connected
call_completed
call_failed
call_result
queue_updated
callback_created
campaign_started
campaign_paused
campaign_completed
tts_error
backend_error

Update only the affected UI components.

Do not reload the entire application unnecessarily.

==================================================
31. DATABASE CONTROL
==================================================

The dashboard must be able to manage database-backed records through backend APIs.

The frontend should NOT directly manipulate PostgreSQL.

Support CRUD where appropriate:

Customers
Companies
Campaigns
Devices
SIMs
Scripts
TTS profiles
Callbacks
Users

For destructive actions:

Show confirmation dialog.

For important actions:

Create audit log.

==================================================
32. SAFETY / AUTHORIZATION
==================================================

This system is for authorized enterprise calling workflows.

Do NOT implement:

- random-number generation
- random dialing
- scraping phone numbers
- bypassing DND/consent systems
- telecom restriction bypass
- CAPTCHA bypass
- unauthorized extraction
- impersonation

Only backend-authorized customer records should enter the calling queue.

A customer marked "Not Interested" must not be repeatedly called contrary to configured/company policy.

==================================================
33. RESPONSIVE DESIGN
==================================================

Desktop:

Sidebar + full dashboard.

Tablet:

Collapsible sidebar.

Mobile:

Navigation drawer.

Tables should become:

- cards
- horizontal scroll where necessary
- responsive detail panels

Do not destroy important information on small screens.

==================================================
34. LOADING / EMPTY / ERROR STATES
==================================================

Every page must implement:

Loading
Empty
Success
Error
Offline
Unauthorized

Examples:

No campaigns yet.

No devices connected.

No callbacks today.

Backend unavailable.

Permission denied.

Do not show blank white screens.

==================================================
35. MOCK MODE
==================================================

The dashboard must work without the Rust backend.

Create mock repositories.

Mock:

Companies
Customers
Campaigns
Devices
SIMs
Calls
Callbacks
Scripts
TTS
Reports
Users

Provide a clearly identifiable Development/Mock Mode.

The UI should behave as if real data exists.

==================================================
36. PROJECT STRUCTURE
==================================================

Use a modular Flutter structure:

lib/
  core/
    config/
    constants/
    network/
    auth/
    routing/
    errors/
    logging/
    theme/

  data/
    models/
    repositories/
    datasources/

  domain/
    entities/
    repositories/
    usecases/

  features/
    dashboard/
    companies/
    customers/
    campaigns/
    devices/
    live_calls/
    call_queue/
    scripts/
    tts/
    callbacks/
    call_history/
    reports/
    users/
    audit_logs/
    settings/

  shared/
    widgets/
    dialogs/
    tables/
    charts/
    status/

Do not create giant Dart files.

Keep modules independent and maintainable.

==================================================
37. PERFORMANCE
==================================================

Optimize for:

- large customer lists
- pagination
- lazy loading
- efficient WebSocket updates
- minimal rebuilds
- cached data
- debounced search
- efficient charts
- responsive UI

Do not load thousands of records into memory unnecessarily.

==================================================
38. SECURITY
==================================================

Do not:

- hard-code credentials
- store passwords
- expose API keys
- connect directly to PostgreSQL
- expose sensitive data unnecessarily

Use:

- secure token storage
- HTTPS
- authenticated API
- role-based UI
- backend authorization
- audit logging

Mask phone numbers where appropriate.

==================================================
39. FINAL UI QUALITY
==================================================

Every screen must be production-quality.

Do NOT generate:

- fake buttons
- empty pages
- TODO placeholders
- broken routes
- inconsistent components
- random styling
- giant widgets
- duplicated business logic

Every visible button must work in mock mode or clearly explain why it is disabled.

==================================================
40. FINAL IMPLEMENTATION REQUIREMENT
==================================================

Do not stop at designing the dashboard.

Implement:

- Flutter project
- routing
- sidebar
- top bar
- all screens
- reusable components
- responsive layouts
- mock data
- state management
- navigation
- forms
- tables
- filters
- dialogs
- charts
- real-time simulation
- loading states
- error states
- empty states
- dark/light theme

Create clean interfaces for future:

Rust REST API
Rust WebSocket
PostgreSQL-backed repositories
Android Calling Agent
TTS providers
STT providers

Do not implement the Rust backend inside this Flutter project.

==================================================
41. FINAL AUDIT
==================================================

Before declaring completion:

Check:

- All routes work
- Sidebar navigation works
- Back navigation works
- All buttons have behavior
- Forms validate input
- Tables render
- Filters work
- Search works
- Mock data works
- Dark mode works
- Responsive layouts work
- Loading states work
- Error states work
- Empty states work
- WebSocket simulation works
- Device status changes update UI
- Campaign workflow works
- Call workflow simulation works
- No giant files
- No TODOs
- No fake credentials
- No broken imports
- No unused critical dependencies
- No direct PostgreSQL connection
- No unauthorized calling mechanism

Fix all issues found during the audit.

==================================================
42. FINAL OUTPUT

At completion provide:

1. Final architecture
2. Route map
3. Screen map
4. User workflow
5. Data flow
6. State-management architecture
7. Mock API/data architecture
8. Folder structure
9. Technology decisions
10. Complete Flutter implementation
11. Setup instructions
12. Backend integration points
13. Android device integration points
14. Testing instructions
15. Final audit report

The final result must be a **complete professional Flutter Control Center**, designed to manage an expandable fleet of Android Calling Agents from:

SIM-1
SIM-2
SIM-3
...
SIM-N

The dashboard must remain company/workflow agnostic at its core and should be configurable for different authorized enterprise calling workflows without rewriting the core application.