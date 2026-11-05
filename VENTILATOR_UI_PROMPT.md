# Ventilator Dashboard — Complete UI Generation Prompt

Yeh prompt kisi bhi AI model (Claude, GPT-4, Gemini) ko diya ja sakta hai
taaki woh ek complete, working ventilator monitoring dashboard prototype
generate kar sake. Single HTML file me, mobile-first, dark theme, demo-grade.

---

## PROMPT (copy from here ↓)

Build a complete, single-file HTML/CSS/JS mobile-first ventilator monitoring
dashboard prototype. Output ek self-contained `.html` file hona chahiye with
inline CSS/JS. Use Tailwind via CDN, Chart.js via CDN, and jsPDF via CDN.
Demo-grade, not for real clinical use. Add clear "DEMO / NOT FOR CLINICAL
USE" disclaimers throughout the UI.

### TECH STACK
- Single HTML file (no build step)
- Tailwind CSS via `https://cdn.tailwindcss.com`
- Chart.js via `https://cdn.jsdelivr.net/npm/chart.js@4.4.0/dist/chart.umd.min.js`
- jsPDF via `https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js`
- Web Audio API for alarm sounds
- localStorage for persistence
- `lang="hi"` on `<html>`, but all UI text in English (Hinglish comments OK)

### LAYOUT
- Mobile-first, max-width `md` (≈448px), full-height column
- Sticky top header: device ID, session ID, operator ID, BLE/Wi-Fi/audio badges, battery %
- Sticky second strip: two buttons — "Scenario" and "Mode"
- Below: alarm banner (red/amber), then main screen content
- Fixed bottom nav with 5 tabs: Home / Trends / Patient / Alerts / More
- Padding-bottom to clear the nav

### SCREENS

**1. Lock screen (initial)**
- 4-digit PIN keypad UI (3×4 grid, plus CLR/DEL)
- 4 dots indicator
- Default PIN: `1234`, shown as a small hint
- On success → unlock main app

**2. Home / Dashboard**
Cards (Tailwind rounded-3xl, border-slate-800, bg-slate-900):
- SpO₂ / HR — large number + circular SVG progress ring + horizontal bar
- Mode / FiO₂ / I:E — 3-col grid
- Delivered BPM + Minute Volume — 2-col
- Tidal Volume + Plateau Pressure — 2-col
- Airway Pressure with PIP/PEEP/MV + pressure waveform canvas
- Flow waveform canvas (L/min)
- Volume waveform canvas (mL)
- Pressure–Volume loop canvas (X = volume, Y = pressure, with trail fill)
- Battery (voltage, current, % bar) + Est. Runtime — 2-col
- Power Source + Motor Temp — 2-col
- Emergency Stop button (red, hold 1.5s → confirmation modal)

**3. Trends**
- Chart.js line chart, last 5 min, 3 series: PIP, PEEP, TV/10
- 4 average tiles: PIP avg, PEEP avg, TV avg, MV avg
- "Export trend as CSV" button

**4. Patient**
- Form: name, ID, age, gender (M/F/O), height (cm), weight (kg)
- "Save Patient" button
- Calculated IBW + TV target range (6–8 mL/kg IBW)
  - IBW male: 50 + 0.91 × (H − 152.4)
  - IBW female: 45.5 + 0.91 × (H − 152.4)

**5. Alerts**
- Active + recent alerts list with level-based colors (info/high/critical)
- "Acknowledge All" + "Clear All" buttons

**6. More (Logs / Device / Settings)**
- Logs section: count, Export JSON / CSV / PDF / FHIR Bundle / Clear
- Device info: ID, serial, hardware, firmware, hours of op, filter status
- "Check for firmware updates" → animated OTA progress modal
- QR onboarding (2 placeholder QRs)
- Settings: dark theme toggle, audio alarm toggle,
  sliders for high-pressure limit, low-SpO₂ threshold, low-battery threshold,
  change PIN, reset demo

### MODALS
- Mode selector (CMV / SIMV / PSV / CPAP / BiPAP) — opens after PIN confirm
- Scenario picker (Normal Adult / ARDS / COPD / Pediatric) — instant apply
- E-stop confirm (red modal with "STOP NOW" button)
- OTA progress modal with animated progress bar
- PIN prompt (re-used for mode change, reset, PIN change)

### BREATH CYCLE (MUST be realistic)
A proper breath cycle has 3 phases inside one cycle (60/RR seconds):
- Inspiration (40%): pressure eases from PEEP to PIP using
  `P = PEEP + (PIP − PEEP) × (1 − cos(πf)) / 2` where f ∈ [0,1]
- Plateau (10%): pressure holds at PIP − 0.2; flow = 0
- Expiration (50%): pressure decays to PEEP using
  `P = PIP − (PIP − PEEP) × (1 − exp(−3f))`; flow is negative and decays

Flow (L/min) is `±TV / inspT * sin/exp shapes`; volume is integrated then
reset. Use this for all 3 scrolling waveforms AND the PV loop.

### ALARMS
- Three levels: info (silent), high (amber tone, ~660Hz square wave),
  critical (red tone, ~880Hz square wave, 2 beeps)
- Audio via Web Audio API; on/off toggle in settings
- Audio badge in header reflects current state
- Alarms auto-clear after 8s; ACK button stops audio immediately
- Thresholds: high pressure, low SpO₂, low battery, hot motor

### SCENARIOS (apply preset vital ranges)
- Normal Adult: SpO₂ 97, HR 74, RR 16, TV 450, PIP 22, PEEP 5, FiO₂ 40
- ARDS:        SpO₂ 92, HR 95, RR 22, TV 350, PIP 32, PEEP 12, FiO₂ 60
- COPD:        SpO₂ 94, HR 88, RR 18, TV 500, PIP 26, PEEP 6,  FiO₂ 35
- Pediatric:   SpO₂ 98, HR 110, RR 24, TV 120, PIP 18, PEEP 4, FiO₂ 30

### EXPORTS
- JSON: full session + device + patient + logs
- CSV: timestamp, device, session, mode, spo2, hr, rr, tv, mv, pip, peep, fio2
- PDF (jsPDF): header + patient + latest vitals + active alerts
- FHIR Bundle (JSON): Patient, Device, Observation (SpO₂/HR/RR) resources
- Trend CSV: timestamp, pip, peep, tv, mv, spo2

### DATA / SIMULATION
- Tick every 1s: random walk each vital within scenario bounds
- Battery drifts down 0.01% per tick (sometimes 0.1%)
- Voltage and current walk within realistic ranges
- Each tick: check thresholds, raise alerts, append to log
- Push chart data every 5 ticks (~5s)

### COLORS
- Background: `bg-slate-950`
- Cards: `bg-slate-900 border border-slate-800 rounded-3xl`
- Primary: `text-sky-300 / bg-sky-500/10`
- Critical: `red-500/40 / red-500/10 / red-300`
- High: `amber-500/40 / amber-500/10 / amber-300`
- OK: `emerald-300/400`
- Text muted: `text-slate-400`, very small: `text-slate-500`, tiny: `text-[10px]`

### UX RULES
- Every value that could be clinically dangerous must have a disclaimer
  ("DEMO ONLY", "CLINICALLY VALIDATED PARAMETER REQUIRED", etc.)
- Audio alarm must respect the user's audio toggle
- Every modal must have a close/cancel button
- Bottom nav must show which screen is active
- All buttons should have hover/active feedback
- Make the demo work offline (no server needed — pure localStorage)

### CODE QUALITY
- Use a single `state` object, persist with `localStorage` under a versioned key
- Pure render functions; no inline DOM hacks
- All IDs in HTML must be unique and referenced from a single `els` map
- Keep `simulateData`, `render`, `renderAlerts`, `renderLogs`, `addLog`,
  `addAlert` cleanly separated
- Add a `// ===` section comment before every major block

### DISCLAIMERS (use these exact strings somewhere visible)
- "PROTOTYPE MONITORING UI"
- "DEMO ONLY — NOT FOR CLINICAL USE"
- "CLINICALLY VALIDATED PARAMETER REQUIRED"
- "UI request only. Real device me hardware safety supervisor zaroori hai."

### DELIVERABLE
Produce ONE self-contained `.html` file (~1500–2000 lines) that opens
directly in a modern browser (Chrome/Edge/Safari) with no server.
Mobile viewport, dark theme by default, light theme toggle in settings.
PIN-gated, audio alarms, full exports, realistic breath waveforms.

---

## NOTES FOR USING THIS PROMPT

- Paste the above "PROMPT" block (between the lines) into Claude / GPT-4 / Gemini
- Attach the original file (`ventilator-dashboard.html`) as context if you want
  the AI to extend it rather than rewrite
- Tell the AI: "Output complete HTML in a single code block, no truncation"
- If output is truncated, ask: "Continue from where you stopped"
- Test by saving as `.html` and opening in browser
- Use Chrome DevTools mobile view (iPhone 12 / Pixel 5) to verify

## WHAT'S IN THE BUNDLED FILE
The file `ventilator-dashboard-full.html` (in this same folder) is the
reference implementation built from this spec — it works as-is in a browser.
