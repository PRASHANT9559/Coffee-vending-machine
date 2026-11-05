# MASTER ARCHITECTURE PROMPT
## JARVIS — Recursive Language Model (RLM) Autonomous Execution System

You are a Principal AI Systems Architect, Recursive Language Model researcher, Agentic AI engineer, Android OS architect, cybersecurity architect, distributed-systems engineer, and MCP/tool-calling specialist.

I am building a JARVIS-like AI-native Android system on:

- Device: OnePlus Nord CE 3 Lite 5G
- Model: CPH2467
- SoC: Qualcomm Snapdragon 695 5G
- RAM: 8 GB
- Storage: 128 GB
- OS foundation: LineageOS
- External AI brain: Google Gemini
- Local persistent memory: encrypted on-device storage
- Tool ecosystem: 245+ Android/OS/application tools
- Architecture goal: RLM-based autonomous agent

IMPORTANT:

DO NOT write production code.

DO NOT write Flutter code.

DO NOT write Rust code.

DO NOT write Kotlin code.

DO NOT write LineageOS source modifications.

DO NOT write shell commands.

Your task is to produce the COMPLETE SYSTEM ARCHITECTURE and TECHNICAL SPECIFICATION for implementing this system later.

============================================================
1. CORE CONCEPT
============================================================

The system must NOT behave like a traditional voice assistant.

Traditional:

User → AI → Answer

Our architecture:

User
 ↓
Gemini
 ↓
RLM Controller
 ↓
Task Decomposition
 ↓
Tool Discovery
 ↓
Tool Selection
 ↓
Permission/Policy Check
 ↓
Tool Execution
 ↓
Observe Result
 ↓
Evaluate Result
 ↓
Need More Information?
 ├── YES → Generate Next Subtask → Tool → Result ↺
 └── NO  → Verify → Complete

The AI must be capable of recursively decomposing a complex task into smaller tasks and dynamically deciding what information, tools, resources or actions are required next.

============================================================
2. DEFINE RLM PRECISELY
============================================================

Explain Recursive Language Models in practical engineering terms.

Clearly distinguish:

- Standard LLM
- Chain-of-thought style reasoning
- ReAct
- Agentic tool calling
- Planner/Executor architecture
- Recursive Language Model
- Multi-agent architecture

Explain exactly which characteristics of RLM we are adopting.

Do NOT assume that simply putting an LLM inside a loop automatically makes it an RLM.

Define the recursive boundary.

============================================================
3. JARVIS SYSTEM ROLES
============================================================

Define these components:

A. GEMINI
B. RLM CONTROLLER
C. TASK MANAGER
D. PLANNER
E. EXECUTOR
F. OBSERVER
G. VERIFIER
H. TOOL REGISTRY
I. TOOL DISCOVERY ENGINE
J. PERMISSION ENGINE
K. POLICY ENGINE
L. LOCAL MEMORY ENGINE
M. CONTEXT ENGINE
N. RESOURCE MANAGER
O. EVENT MANAGER
P. SECURITY ENGINE
Q. AUDIT ENGINE
R. HUMAN CONFIRMATION ENGINE
S. ERROR RECOVERY ENGINE
T. LOOP GOVERNOR

Explain the exact responsibility of every component.

============================================================
4. RLM EXECUTION LOOP
============================================================

Design the complete recursive execution loop.

Use this conceptual cycle:

OBSERVE
→ UNDERSTAND
→ PLAN
→ DECOMPOSE
→ SELECT TOOL
→ CHECK PERMISSION
→ EXECUTE
→ OBSERVE RESULT
→ EVALUATE
→ VERIFY
→ REPLAN
→ COMPLETE

The loop must be dynamic.

Example:

User:
"Free up 10 GB of storage without deleting anything important."

The agent should be capable of:

1. Inspect storage
2. Identify large files
3. Classify files
4. Inspect duplicates
5. Inspect caches
6. Check user memory/preferences
7. Generate safe candidates
8. Ask for confirmation if necessary
9. Execute approved operations
10. Verify freed space
11. If target is not reached, recursively investigate another safe strategy
12. Stop when goal is achieved or impossible

The agent must NOT assume the solution before observing the environment.

============================================================
5. SELF-GENERATED SUBTASKS
============================================================

This is a critical requirement.

The AI must be able to generate its own subtasks based on the current state.

Example:

Goal:
"Prepare my phone for sleep."

The agent may dynamically create:

check_current_time()
check_alarm_state()
check_battery()
check_calendar()
check_dnd()
check_volume()
check_brightness()

Then evaluate the results and decide what should happen next.

Explain:

- Parent task
- Child task
- Grandchild task
- Recursive task
- Dependency
- Completion condition
- Failure condition

Design a task tree / task graph model.

============================================================
6. SELF-DIRECTED RESOURCE DISCOVERY
============================================================

The agent must be able to determine:

"I don't have enough information to continue."

It should then search its available resources.

Possible resources:

- Android system APIs
- LineageOS services
- Tool registry
- Local files
- Local memory
- Application data exposed through permitted interfaces
- Device sensors
- Network
- External APIs
- Other tools
- Previous task results

Example:

User:
"Why is my phone overheating?"

Agent:

Need temperature
→ thermal tool

Need CPU usage
→ CPU tool

Need active processes
→ process information tool

Need battery state
→ battery tool

Need recent activity
→ context/memory tool

Then recursively combine the results.

============================================================
7. TOOL DISCOVERY
============================================================

The agent must NOT have to know every tool beforehand.

Design:

Tool Registry
→ capability metadata
→ semantic search
→ candidate tools
→ tool selection
→ schema validation
→ execution

Define how Gemini/RLM discovers tools.

For every tool define metadata such as:

- tool_id
- capability
- description
- input_schema
- output_schema
- permissions
- risk_level
- dependencies
- availability
- reversibility
- confirmation_required

============================================================
8. 245+ TOOLS
============================================================

Integrate the previously designed 245+ Android tools into the architecture.

Do NOT simply list them again.

Instead classify them into:

- Read-only tools
- Basic action tools
- Advanced action tools
- Cross-app tools
- System tools
- Hardware tools
- Communication tools
- File tools
- Automation tools
- Security tools
- Diagnostic tools
- AI/memory tools

Explain how RLM dynamically selects among them.

============================================================
9. LOCAL PERSISTENT MEMORY
============================================================

Design a complete local memory subsystem.

The memory must remain on the user's device.

The memory should NOT depend on Gemini's cloud memory.

Use conceptual layers:

L0 Working Memory
L1 Session Memory
L2 Short-Term Memory
L3 Long-Term User Memory
L4 Episodic Memory
L5 Semantic Memory
L6 Procedural Memory
L7 Device Memory
L8 Automation Memory
L9 Project/Knowledge Memory

The user must be able to:

- View memory
- Search memory
- Edit memory
- Delete memory
- Lock memory
- Export memory
- Import memory
- Mark memory as permanent
- Set expiration
- Disable memory categories

Explain:

Memory ingestion
→ classification
→ deduplication
→ importance scoring
→ confidence scoring
→ storage
→ retrieval
→ context injection
→ update

============================================================
10. MEMORY + RLM
============================================================

Show exactly how RLM uses local memory.

Example:

User:
"Set up my phone like I normally use it."

RLM:

Retrieve preferences
→ identify routines
→ inspect current device state
→ compare expected vs actual
→ generate actions
→ execute
→ verify

The agent must not blindly trust memory.

Memory should be treated as contextual evidence and verified against current device state.

============================================================
11. GEMINI + RLM
============================================================

Gemini is the primary reasoning model.

The local system is the execution environment.

Define the boundary:

Gemini:
- Understands natural language
- Reasons about goals
- Creates plans
- Generates subtasks
- Chooses tools
- Interprets results

Local system:
- Executes tools
- Enforces permissions
- Protects data
- Maintains memory
- Verifies actions
- Controls Android

Gemini must NEVER receive unrestricted root access.

============================================================
12. MCP-STYLE TOOL GATEWAY
============================================================

Design an MCP-inspired architecture.

Include:

Tool Discovery
Tool Schema
Tool Invocation
Tool Result
Tool Errors
Tool Versioning
Capability Discovery
Permission Validation
Policy Validation
Audit Logging

Clearly distinguish:

MCP protocol concepts
from
Android OS authority.

Do NOT claim MCP itself grants Android system privileges.

============================================================
13. RECURSIVE TOOL CALLING
============================================================

Design how the agent can perform:

Tool
→ inspect result
→ decide next tool
→ execute
→ inspect
→ decide next tool

Example:

get_storage()
→ result

search_large_files()
→ result

classify_files()
→ result

find_duplicates()
→ result

calculate_safe_cleanup()
→ result

request_confirmation()
→ result

delete_selected_files()
→ result

verify_storage()
→ result

This must be represented as a controlled recursive execution graph.

============================================================
14. LOOP GOVERNOR
============================================================

This is mandatory.

The RLM must NEVER loop forever.

Design:

- Maximum recursion depth
- Maximum iterations
- Maximum execution time
- Tool-call budget
- Token budget
- Memory retrieval budget
- Network budget
- Energy/battery budget
- Risk budget
- Failure threshold
- Repeated-action detection
- Circular dependency detection
- Deadlock detection
- Emergency stop

Define what happens when a limit is reached.

============================================================
15. SELF-CORRECTION
============================================================

The agent must be capable of recognizing:

- Wrong tool
- Wrong assumption
- Unexpected result
- Partial success
- Failed action
- Contradictory information
- Missing resource

Then:

STOP
→ ANALYZE FAILURE
→ GENERATE ALTERNATIVE
→ RETRY SAFELY
→ VERIFY

But do NOT allow unlimited retries.

============================================================
16. RESULT VERIFICATION
============================================================

Every important action should have a verification strategy.

Example:

set_wifi(true)
→ get_wifi_state()
→ confirm actual state

create_alarm()
→ read_alarm()
→ verify alarm exists

delete_file()
→ search_file()
→ verify deletion

send_message()
→ verify delivery state where technically available

Never report success based only on the tool invocation response.

============================================================
17. HUMAN-IN-THE-LOOP
============================================================

Create a complete confirmation model.

R0:
No confirmation

R1:
Normally no confirmation

R2:
Context-dependent confirmation

R3:
Explicit confirmation

R4:
Biometric / strong confirmation

Examples:

Turn flashlight ON → R1

Read notification → R2

Send message → R3

Delete important files → R3/R4

Security credential changes → R4

Financial transaction → R4

Define the policy engine.

============================================================
18. SECURITY
============================================================

Design a defense-in-depth model.

Include:

- Capability-based security
- Least privilege
- Sandboxing
- SELinux
- Permission broker
- Tool allowlists
- Parameter validation
- Input sanitization
- Prompt injection defense
- Tool poisoning defense
- Memory poisoning defense
- Sensitive data filtering
- Audit logs
- Rate limiting
- Session isolation
- Secure IPC
- Encryption at rest
- Android Keystore
- Secure deletion where practical

Explain how malicious content from a webpage, message, document or application cannot trick Gemini into gaining unauthorized device control.

============================================================
19. MEMORY SECURITY
============================================================

Design memory security separately.

Threats:

- Memory poisoning
- Malicious instructions stored as memories
- Sensitive data leakage
- Unauthorized memory modification
- Cross-user leakage
- AI provider access
- Local malware access

Important:

A stored memory must NEVER automatically become an executable instruction.

Separate:

FACT
PREFERENCE
EVENT
KNOWLEDGE
PROCEDURE
INSTRUCTION
UNTRUSTED CONTENT

============================================================
20. CONTEXT ENGINE
============================================================

Design a context engine combining:

Current user request
+
Current screen/app
+
Device state
+
Recent events
+
Relevant local memory
+
Available tools
+
Permissions
+
Security policy
+
Task history

into a controlled context supplied to Gemini.

Do NOT send unnecessary private data to the cloud model.

============================================================
21. AUTONOMOUS JARVIS BEHAVIOR
============================================================

Design proactive behavior.

Examples:

- Battery anomaly
- Storage anomaly
- Thermal anomaly
- Suspicious application behavior
- Upcoming calendar event
- Repeated routine
- Network changes
- Device health changes

Architecture:

OBSERVE
→ DETECT
→ REASON
→ SUGGEST
→ ASK
→ EXECUTE
→ VERIFY

Define which actions may be automatic and which require confirmation.

============================================================
22. LONG-RUNNING TASKS
============================================================

Design tasks that take minutes/hours.

Example:

"Every night, monitor my storage and tell me if safe cleanup candidates exceed 5 GB."

The system must support:

- Task persistence
- Scheduling
- Wake-up
- State checkpointing
- Pause
- Resume
- Cancellation
- Failure recovery
- Notification
- Audit trail

============================================================
23. INTERRUPTION AND RESUMPTION
============================================================

The user may interrupt the agent.

Example:

"Stop."

The agent must immediately stop eligible actions.

Then:

"Continue what you were doing."

The agent should restore the task from a safe checkpoint.

Design checkpoint structure.

============================================================
24. MULTI-STEP EXAMPLE SCENARIOS
============================================================

Provide at least 20 realistic scenarios.

Include:

1. Phone optimization
2. Storage cleanup
3. Travel preparation
4. Meeting preparation
5. Morning routine
6. Sleep routine
7. Notification management
8. File organization
9. Document processing
10. Media organization
11. Battery optimization
12. Device troubleshooting
13. Calendar management
14. Communication workflow
15. Cross-app task
16. Network troubleshooting
17. Security investigation
18. PC/device interaction
19. IoT automation
20. Complex JARVIS-style autonomous task

For each scenario show:

User Goal
→ RLM Plan
→ Recursive Subtasks
→ Tools
→ Results
→ Replanning
→ Verification
→ Completion

============================================================
25. MERMAID ARCHITECTURE
============================================================

Produce detailed Mermaid diagrams for:

A. Overall architecture
B. RLM execution loop
C. Task decomposition
D. Tool discovery
E. Tool execution
F. Local memory
G. Gemini integration
H. Permission engine
I. Security architecture
J. Human confirmation
K. Error recovery
L. Long-running task
M. Autonomous JARVIS flow

============================================================
26. DATA MODELS
============================================================

Design conceptual schemas for:

Task
Subtask
Tool
ToolCall
ToolResult
Memory
Context
Permission
Policy
Confirmation
Checkpoint
Execution
AuditEvent
AgentSession

Do NOT write database code.

============================================================
27. TECHNOLOGY MAPPING
============================================================

Map every subsystem to possible implementation technology.

Consider:

Flutter
Rust
Kotlin
Android Framework
LineageOS
SQLite/SQLCipher or equivalent encrypted local database
Android Keystore
SELinux
IPC
gRPC where appropriate
Gemini API
MCP-compatible gateway
Vector/semantic retrieval where useful

For each technology explain:

Why
Where
Advantages
Disadvantages
Security implications

============================================================
28. OFFLINE / ONLINE MODES
============================================================

Design:

ONLINE MODE
Gemini available

LIMITED OFFLINE MODE
Cloud model unavailable but local tools remain available

FULL OFFLINE MODE
Local lightweight model if later added

Explain how the same Tool Registry and Memory system can work regardless of which AI model is being used.

The architecture must avoid vendor lock-in.

============================================================
29. MODEL AGNOSTIC ARCHITECTURE
============================================================

Gemini is the initial model.

But the architecture must allow future models.

Examples:

Gemini
Local model
Qwen
Llama
Other compatible models

The Tool Registry, Memory, Security and Execution Engine must remain independent from the model provider.

============================================================
30. JARVIS CONTROL PHILOSOPHY
============================================================

The system should behave according to:

USER OWNS THE DEVICE
AI HAS CAPABILITIES
POLICY DEFINES AUTHORITY
TOOLS PERFORM ACTIONS
MEMORY PROVIDES CONTEXT
RLM PROVIDES ADAPTIVE EXECUTION
VERIFICATION PROVES RESULTS

AI intelligence must never equal unlimited authority.

============================================================
31. FINAL MASTER ARCHITECTURE
============================================================

At the end produce one complete architecture:

USER
 ↓
VOICE/TEXT INTERFACE
 ↓
CONTEXT ENGINE
 ↓
GEMINI
 ↓
RLM CONTROLLER
 ↓
TASK GRAPH
 ↓
MEMORY RETRIEVAL
 ↓
TOOL DISCOVERY
 ↓
POLICY ENGINE
 ↓
PERMISSION ENGINE
 ↓
HUMAN CONFIRMATION
 ↓
TOOL EXECUTOR
 ↓
ANDROID / LINEAGEOS
 ↓
RESULT
 ↓
OBSERVER
 ↓
VERIFIER
 ↓
RLM
 ↺

Also show where:

- Local Memory
- Security Engine
- Audit Engine
- Loop Governor
- Error Recovery
- Scheduler
- Event Manager

fit into the architecture.

============================================================
32. CRITICAL ENGINEERING QUESTIONS
============================================================

Before finalizing, answer these questions:

1. What exactly makes this architecture recursive?
2. Where does recursion occur?
3. How does the agent decide that it needs another resource?
4. How does it discover that resource?
5. How does it decide which tool to call?
6. How does it know when to stop?
7. How does it recover from failure?
8. How does it prevent infinite loops?
9. How does local memory influence planning?
10. How do we prevent memory poisoning?
11. How does Gemini interact with Android without root access?
12. Which capabilities require LineageOS modification?
13. Which require privileged Android APIs?
14. Which require Accessibility?
15. Which require root?
16. Which capabilities are impossible/restricted?
17. How do we maintain security while providing high autonomy?
18. How do we make the architecture model-agnostic?
19. How can the architecture scale from 245 tools to 1,000+ tools?
20. How can a new tool be added without changing the core agent?

============================================================
33. FINAL OUTPUT REQUIREMENTS
============================================================

Structure your answer exactly as:

PART 01 — Executive Summary
PART 02 — RLM Definition
PART 03 — System Components
PART 04 — RLM Execution Loop
PART 05 — Recursive Task Graph
PART 06 — Dynamic Resource Discovery
PART 07 — Tool Registry Architecture
PART 08 — 245+ Tool Integration
PART 09 — Local Persistent Memory
PART 10 — Memory + RLM
PART 11 — Gemini Integration
PART 12 — MCP-Style Gateway
PART 13 — Permission Architecture
PART 14 — Security Architecture
PART 15 — Loop Governor
PART 16 — Verification System
PART 17 — Self-Correction
PART 18 — Human-in-the-Loop
PART 19 — Context Engine
PART 20 — Proactive JARVIS
PART 21 — Long-Running Tasks
PART 22 — Interruption / Resume
PART 23 — 20+ Agentic Scenarios
PART 24 — Data Models
PART 25 — Technology Architecture
PART 26 — Offline/Online Architecture
PART 27 — Model-Agnostic Architecture
PART 28 — Mermaid Diagrams
PART 29 — Threat Model
PART 30 — Critical Engineering Questions
PART 31 — Complete JARVIS Master Architecture
PART 32 — Recommended Implementation Roadmap

============================================================
FINAL INSTRUCTION
============================================================

Think beyond a voice assistant.

Design this as the foundation of an AI-native operating environment where a powerful external model can recursively reason about a user's goal, discover the resources it needs, dynamically call tools, inspect results, generate new subtasks, recover from failures, use persistent local memory, verify outcomes, and continue until the task is complete.

However:

AUTONOMY MUST NEVER MEAN UNLIMITED AUTHORITY.

The final architecture must be:

- Production-oriented
- Security-first
- Recursive
- Tool-driven
- Memory-driven
- Model-agnostic
- Android-aware
- LineageOS-compatible
- Extensible
- Auditable
- Fault-tolerant
- Human-controlled

Do not produce implementation code.

Produce the deepest possible architecture and engineering specification.