# ROLE

You are the **Senior Software Architect, Product Engineer, UI/UX Engineer, Automation Engineer, AI/Voice Engineer, QA Engineer, Security Engineer and DevOps Engineer** for this project.

I am the junior developer/product owner.

Do NOT ask me to design the architecture, choose the technology, decide the workflow, or solve engineering problems that you can reasonably solve yourself.

You must make those decisions as the senior engineer.

Your goal is to deliver a **100% working, production-ready project**, not a demo, prototype, pseudo-code project, or partially implemented skeleton.

---

# PROJECT

Build a complete **Excel-driven Hindi Call-Center Automation / Agent-Assistance System**.

The system receives customer information from an Excel/CSV file and automates as much of the repetitive call-center workflow as technically and legally possible.

Example input:

Name: Prashant
Mobile: XXXXXXXX10
Recharge_Status: Expired
Offer: ₹299
Validity: 28 Days
Benefits: Unlimited calling, 100 SMS daily and more

The system should generate a personalized Hindi calling script automatically.

Example:

"नमस्ते Prashant जी, आपके मोबाइल नंबर पर रिचार्ज समाप्त हो गया है। अभी आपके नंबर पर ₹299 का 28 दिनों वाला ऑफर उपलब्ध है, जिसमें अनलिमिटेड कॉलिंग और 100 SMS प्रतिदिन की सुविधा है। अगर आप रिचार्ज करना चाहते हैं तो 1 दबाएँ। बाद में कॉल करने के लिए 2 दबाएँ।"

---

# IMPORTANT LEGAL/SYSTEM BOUNDARY

This software must NOT:

- scrape random phone numbers from the internet
- generate random customer numbers
- call people without authorization
- bypass telecom restrictions
- bypass CAPTCHA, authentication, security controls or company restrictions
- extract data from a company system without authorization
- impersonate a telecom company
- hide the identity/purpose of the call
- bypass Do-Not-Call/consent/compliance requirements

The system must operate only on customer data that the operator/company is authorized to process.

For actual automated outbound calls, design the system around a **legitimate/authorized telephony provider or company-approved calling infrastructure**.

If a particular integration requires credentials/API access, implement the integration layer cleanly and provide configuration placeholders instead of inventing credentials.

---

# PRIMARY OBJECTIVE

Reduce repetitive manual work by approximately 70–90% where technically possible.

The system should automate:

1. Excel/CSV import
2. Data validation
3. Customer queue generation
4. Personalized Hindi script generation
5. Hindi text-to-speech
6. Call initiation through an authorized telephony integration
7. Customer response detection
8. DTMF input such as 1/2/3
9. Speech-to-text where supported
10. Basic intent classification
11. Call result classification
12. Callback scheduling
13. Call history
14. Excel/CSV status update
15. Error handling
16. Retry handling
17. Duplicate prevention
18. Dashboard/statistics
19. Logs
20. Configuration management

---

# YOU DECIDE THE ARCHITECTURE

Do NOT ask me:

- Which programming language?
- Which database?
- Which TTS?
- Which speech recognition system?
- Which telephony provider?
- Which frontend framework?
- Which backend framework?
- Which folder structure?
- Which architecture pattern?

Evaluate the requirements and choose the best stack yourself.

Prefer:

- reliability
- low resource usage
- maintainability
- low operational complexity
- modular architecture
- privacy
- security
- easy local development
- production deployment capability

If a component should be local/offline, choose local processing.

If a cloud/API service is genuinely required, clearly isolate it behind an interface.

---

# ARCHITECTURE REQUIREMENTS

Create a clean modular architecture containing appropriate modules such as:

- Excel/CSV Import
- Customer Data Model
- Validation Engine
- Campaign/Queue Manager
- Script Generator
- Hindi TTS Engine
- Telephony Adapter
- Call Session Manager
- DTMF Handler
- Speech-to-Text Adapter
- Intent Detection
- Response Handler
- Callback Scheduler
- Call History
- Excel Export/Update
- Dashboard
- Configuration
- Logging
- Security
- Error Recovery
- Testing

You may rename, merge, split or replace these modules if your architecture is better.

---

# INPUT FORMAT

Support at minimum:

```text
Name
Mobile
Recharge_Status
Offer
Validity
Benefits
Call_Status
```

Also support optional:

```text
Customer_ID
Language
Preferred_Call_Time
Last_Call_Time
Max_Attempts
Remarks
Callback_Time
```

The system must tolerate:

- missing optional fields
- extra columns
- empty rows
- duplicate numbers
- invalid numbers
- Unicode/Hindi text
- ₹ currency symbols
- different Excel column ordering

---

# CALL WORKFLOW

Implement this workflow:

```text
Import Excel
    ↓
Validate Data
    ↓
Remove/flag invalid records
    ↓
Create Calling Queue
    ↓
Select next eligible customer
    ↓
Generate personalized Hindi script
    ↓
Generate/play Hindi voice
    ↓
Initiate authorized call
    ↓
Wait for customer
    ↓
Detect DTMF / speech response
    ↓
Classify response
    ↓
Take appropriate action
    ↓
Update customer record
    ↓
Save call log
    ↓
Move to next customer
```

---

# CUSTOMER RESPONSE LOGIC

At minimum support:

### 1
Customer wants the offer/recharge.

Result:

```text
Interested
```

### 2
Customer wants a callback.

Result:

```text
Callback
```

Store callback time if available.

### 3
Customer does not want the offer.

Result:

```text
Not Interested
```

### No response

Result:

```text
No Answer
```

### Busy

Result:

```text
Busy
```

### Failed call

Result:

```text
Failed
```

### Unknown speech

Result:

```text
Unknown
```

Do not make dangerous assumptions from ambiguous speech.

---

# HINDI VOICE

The system must support natural Hindi speech.

Create a TTS abstraction:

```text
TTSProvider
    ├── ProviderA
    ├── ProviderB
    └── LocalProvider (if practical)
```

Do not hard-code the entire application to one vendor.

The user must be able to configure the selected provider.

Handle:

- Hindi
- English numbers
- ₹ amounts
- customer names
- dates
- validity
- offer information

correctly.

---

# SPEECH RECOGNITION

Create a speech-recognition abstraction.

The system should recognize common Hindi responses such as:

- हाँ
- नहीं
- बाद में फोन करना
- अभी नहीं
- रिचार्ज करना है
- ऑफर बताइए
- कल फोन करना
- शाम को फोन करना
- interested
- no
- later
- yes
- no

Do not rely only on exact string matching.

Use an intent layer.

Example:

```text
"हाँ कर दीजिए"
→ Interested

"अभी व्यस्त हूँ, शाम को फोन करना"
→ Callback

"नहीं चाहिए"
→ Not Interested
```

---

# EXCEL UPDATE

After every call, update the corresponding record.

At minimum:

```text
Call_Status
Call_Time
Call_Duration
Customer_Response
Intent
Callback_Time
Attempt_Count
Remarks
```

Never corrupt the original source file.

Use a safe strategy such as:

```text
input.xlsx
processed_output.xlsx
```

or another architecture you consider safer.

Implement backup/recovery where appropriate.

---

# UI

Build a clean operator dashboard.

It should show:

```text
Total Customers
Pending
Called
Interested
Not Interested
Callback
No Answer
Failed
```

Also show:

- current customer
- call status
- current script
- call timer
- detected response
- next customer
- callback queue
- error messages
- import/export controls
- settings

The UI must be simple enough for a normal call-center employee.

---

# SECURITY

Implement:

- input validation
- phone-number validation
- secrets outside source code
- environment variables/configuration
- safe logging
- no unnecessary storage of sensitive data
- access control where appropriate
- encrypted transport for network communication
- secure API credential handling
- audit logging

Never print API keys or credentials into logs.

Mask phone numbers in logs where appropriate.

---

# ERROR HANDLING

Handle:

- Excel file locked
- malformed Excel
- invalid phone number
- duplicate customer
- TTS failure
- speech recognition failure
- telephony API failure
- network timeout
- customer disconnect
- provider rate limit
- API authentication failure
- application crash
- computer restart
- partially completed campaign

The system must recover gracefully.

---

# RETRY POLICY

Design a sensible retry system.

Example:

```text
No Answer
    ↓
Retry according to configured policy

Busy
    ↓
Retry later

Failed
    ↓
Retry only when appropriate

Not Interested
    ↓
Do NOT automatically keep calling
```

Do not create aggressive repeated calling behavior.

Make retry limits configurable.

---

# DATA MODEL

Design proper data structures/database schema if a database is useful.

At minimum represent:

```text
Customer
Campaign
CallAttempt
CallResult
Callback
Configuration
AuditLog
```

Choose SQL/SQLite/PostgreSQL/etc. yourself based on the architecture.

Do not introduce a database if a simpler reliable architecture is sufficient.

---

# OFFLINE / ONLINE DESIGN

Separate the system into:

```text
Core Application
        ↓
Provider Interfaces
        ↓
External Services
```

The application should remain testable without real telephony credentials.

Create mock implementations for:

- TTS
- Speech Recognition
- Telephony

This allows complete local testing.

---

# TEST MODE

This is mandatory.

Create a simulation mode where:

```text
Excel
 ↓
Script
 ↓
Mock Call
 ↓
Simulated Customer Response
 ↓
Intent Detection
 ↓
Excel Update
```

No real phone call should happen in test mode.

Provide test cases for:

- Interested
- Not Interested
- Callback
- No Answer
- Busy
- Failed
- Unknown response

---

# PROJECT QUALITY REQUIREMENT

Do not give me:

- pseudo-code
- TODO placeholders
- "implement this later"
- fake APIs
- fake credentials
- incomplete screens
- empty classes
- dummy business logic
- broken imports
- unimplemented functions
- architecture-only documentation

If an external service cannot be used without credentials, implement the complete adapter/interface and configuration system and make the rest of the application fully runnable using mocks.

---

# DEVELOPMENT PROCESS

Work like a senior engineer.

First internally determine:

1. Requirements
2. Constraints
3. Architecture
4. Technology stack
5. Data model
6. Workflow
7. UI
8. Integration strategy
9. Security model
10. Testing strategy
11. Deployment strategy

Then implement the project.

Do not repeatedly ask me to make engineering decisions that you can make yourself.

If there are multiple technically valid options, choose the one with the best balance of reliability, simplicity, cost, maintainability and performance.

Explain important architectural decisions briefly after implementation.

---

# FOLDER STRUCTURE

Create a professional project structure.

Do not put the entire project into one huge file.

Separate:

- UI
- business logic
- integrations
- data layer
- models
- services
- utilities
- tests
- configuration

Keep individual source files reasonably sized and maintainable.

---

# DOCUMENTATION

Create:

```text
README
ARCHITECTURE
WORKFLOW
SETUP
CONFIGURATION
API_INTEGRATION
TESTING
SECURITY
TROUBLESHOOTING
```

Documentation must describe the actual implementation, not an imaginary future implementation.

---

# FINAL AUDIT

Before declaring the project complete, perform a senior-level audit:

### Functional
- Does the application start?
- Can Excel be imported?
- Is data validated?
- Is Hindi script generated?
- Does test-mode calling work?
- Are responses classified?
- Is Excel updated?
- Are callbacks stored?
- Are logs generated?

### Reliability
- Are failures handled?
- Are duplicate calls prevented?
- Are retries controlled?
- Can the application recover after restart?

### Security
- Are secrets protected?
- Is sensitive data handled safely?
- Are unauthorized calling mechanisms absent?

### Code Quality
- Are there dead files?
- Broken imports?
- TODOs?
- Fake implementations?
- Unused dependencies?
- Hard-coded secrets?
- Excessively large files?
- Duplicate business logic?

Fix every issue you find.

---

# IMPORTANT

Do not stop after generating an architecture.

Do not stop after generating code.

Do not tell me "now you need to implement X".

**You are responsible for completing the entire project.**

If something is impossible without an external credential or company-specific API, make the rest of the project fully functional using a mock/test adapter and clearly isolate the external integration point.

At the end provide:

1. Final architecture
2. Final workflow
3. Technology choices and why
4. Complete project implementation
5. Setup instructions
6. Configuration instructions
7. Test instructions
8. Production deployment instructions
9. External credentials/API requirements
10. Final audit report
11. Known limitations

Treat me as the junior developer and yourself as the senior engineer responsible for the technical outcome.