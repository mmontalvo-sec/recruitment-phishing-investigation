# Detection Opportunities

**Case:** RPI-2026-001
**Purpose:** Where security controls could have flagged this campaign earlier

This document maps points in the attack chain where available security tooling, if configured or present, could have generated an alert or blocked the attack. Each entry includes the attack phase, the detection opportunity, the tool or control that could implement it, and a confidence level reflecting how reliably the technique would have fired.

---

## Phase 1: Email Delivery

### D-001 — Sender Domain Reputation Check

**What to detect:** Email from `officesdesknote[.]com` arriving in a personal or corporate inbox
**Detection method:** Email security gateway (Proofpoint, Defender for Office 365, Google Workspace Protect) with sender domain age and reputation scoring
**Signal:** `officesdesknote[.]com` was not associated with Mercer Bucks Technology and had no established sending reputation
**Confidence:** Medium (depends on domain age at time of campaign; newly registered domains often score poorly)

---

### D-002 — Display Name / Domain Mismatch Alert

**What to detect:** Email claiming to represent a company while sending from a non-matching domain
**Detection method:** Email security gateway rule: display name contains known company name but sender domain does not match verified company domains
**Signal:** Display name "Tammy Dotson / Mercer Bucks Technology" but sending domain was `officesdesknote[.]com`, not `mercerbuckstechnology.com` or similar
**Confidence:** High (this is a standard impersonation detection rule available in most enterprise email security platforms)

---

## Phase 2: Link Click and Page Visit

### D-003 — Safe Links / URL Scanning

**What to detect:** Click on a URL that resolves to a known phishing domain or that contains suspicious patterns
**Detection method:** Microsoft Defender Safe Links, Proofpoint URL Defense, browser extension (uBlock Origin, Malwarebytes Browser Guard)
**Signal:** `ms-teaminvite[.]team` was not a Microsoft domain and would have been flagged as suspicious by domain category analysis. By June 5 it was confirmed as phishing by Cloudflare.
**Confidence:** Medium-High (depends on whether the domain had been categorized at time of click)

---

### D-004 — Browser Warning on Camera/Mic Permission Request from Unknown Domain

**What to detect:** Non-Microsoft domain requesting camera and microphone access under a Microsoft Teams framing
**Detection method:** User awareness; browser security UX (the browser clearly showed `ms-teaminvite.team` in the address bar during the permission prompt)
**Signal:** The permission prompt from the browser explicitly listed the requesting domain as `ms-teaminvite.team`, not `teams.microsoft.com`
**Confidence:** High if user is trained to check the domain in permission prompts; Low without that training

---

## Phase 3: File Download

### D-005 — Browser Warning on .js File Download

**What to detect:** Download of a `.js` file from a non-trusted domain
**Detection method:** Browser download filtering (Chrome, Edge, Brave all warn on executable-type downloads from unverified sites); SmartScreen at download time
**Signal:** Brave browser may have presented a download warning; the `.js` file extension itself is a high-risk download type
**Confidence:** Medium (depends on browser security settings and whether SmartScreen was active)

---

### D-006 — Endpoint File Extension Policy Enforcement

**What to detect:** `.js` file present in Downloads or Desktop executed by Windows Script Host
**Detection method:** Windows Group Policy or AppLocker rule blocking `wscript.exe` and `cscript.exe` from executing user-downloaded `.js` files; Software Restriction Policies
**Signal:** `wscript.exe MS-TeamSetup.js` or `cscript.exe MS-TeamSetup.js` in process telemetry
**Confidence:** High if policy is in place; this control would have completely blocked execution

---

## Phase 4: Script Execution

### D-007 — ActiveX Object Creation via WScript.Shell / ADODB.Stream

**What to detect:** Script engine creating `ADODB.Stream` or `WinHttp.WinHttpRequest` objects, especially followed by binary file writes
**Detection method:** EDR telemetry (CrowdStrike, SentinelOne, Microsoft Defender for Endpoint); PowerShell Script Block Logging equivalent for WSH
**Signal:** `wscript.exe` spawning HTTP requests and writing binary data to `%TEMP%\` is unusual behavior for a scripting host
**Confidence:** High with EDR deployed; not detectable on a standard consumer Windows installation without additional tooling

---

### D-008 — Outbound Connection to api.ipify.org / icanhazip.com from Non-Browser Process

**What to detect:** Script or non-browser process querying public IP lookup services
**Detection method:** Firewall or DNS monitoring with process attribution; EDR network telemetry
**Signal:** `wscript.exe` or any script engine reaching out to `api.ipify.org` is anomalous; this is a common malware reconnaissance pattern
**Confidence:** High with network-aware EDR; Low without process-attributed network monitoring

---

### D-009 — Outbound Connection to api.telegram.org from Non-Telegram Process

**What to detect:** Script engine or unexpected process communicating with Telegram Bot API
**Detection method:** Firewall application control; EDR network telemetry with process attribution
**Signal:** `wscript.exe` connecting to `api.telegram.org` is not normal behavior; only Telegram Desktop, Telegram Web, or similar user applications should initiate Telegram API traffic
**Confidence:** High with process-attributed network monitoring; requires EDR or next-gen firewall

---

### D-010 — MSI Download to TEMP Directory Followed by msiexec.exe Execution

**What to detect:** `msiexec.exe /i %TEMP%\*.msi` execution, especially when the MSI was recently written to TEMP by a non-standard process
**Detection method:** EDR process chain analysis; Windows Event ID 1040/1042 (MSI install start/end); Defender for Endpoint behavioral alerts
**Signal:** The process chain `wscript.exe → msiexec.exe /i %TEMP%\Microsoft Teams.msi` is not a legitimate Teams installation pattern (Teams installs from an official Teams setup URL, not TEMP)
**Confidence:** High with EDR behavioral rules; not reliably detectable on standard Windows Defender alone

---

## Phase 5: Payload Activity

### D-011 — ScreenConnect Service or DLL Installation

**What to detect:** ScreenConnect/ConnectWise Control client installation creating a new service or DLL under `Program Files (x86)`
**Detection method:** Windows Event Logs (Service Control Manager event 7045 - New Service Installed); EDR telemetry; file integrity monitoring
**Signal:** A ScreenConnect service appearing unexpectedly without IT-initiated deployment
**Confidence:** High if service installation monitoring is in place

---

### D-012 — New Folder Creation in Documents with Security-Themed Name

**What to detect:** Folder `WinSec` (or similar names like `WinLog`, `SysAuth`) created under `%USERPROFILE%\Documents\`
**Detection method:** EDR file system monitoring; file integrity monitoring tools
**Signal:** Legitimate Windows security logs do not reside in the user's Documents folder. New folders with security-mimicking names in user profile directories warrant investigation.
**Confidence:** Medium (high false positive potential without baselining, but easy to filter once a baseline is established)

---

### D-013 — Log Files with PIN-Pattern Content Written to User Directory

**What to detect:** Text files matching pattern `log_PIN_*.txt` appearing in user profile directories
**Detection method:** EDR file creation telemetry; DLP with filename pattern matching
**Signal:** File naming convention combined with content structure (incremental numeric input logging) is highly anomalous and does not match any legitimate Windows application behavior
**Confidence:** High for pattern-matched detection; requires EDR or file monitoring agent

---

## Summary Table

| ID | Phase | Detection Control | Confidence | Control Type |
|---|---|---|---|---|
| D-001 | Email Delivery | Sender domain reputation | Medium | Email Security |
| D-002 | Email Delivery | Display name / domain mismatch | High | Email Security |
| D-003 | Link Click | Safe Links / URL scanning | Medium-High | Email / Browser |
| D-004 | Page Visit | Camera/mic permission domain check | High (trained user) | User Awareness |
| D-005 | File Download | Browser .js download warning | Medium | Browser Security |
| D-006 | File Download | WSH execution block via Group Policy | High | Endpoint Policy |
| D-007 | Script Execution | ActiveX / ADODB.Stream monitoring | High | EDR |
| D-008 | Script Execution | Non-browser IP lookup outbound | High | EDR / Firewall |
| D-009 | Script Execution | Non-Telegram process → Telegram API | High | EDR / Firewall |
| D-010 | Script Execution | TEMP MSI → msiexec process chain | High | EDR |
| D-011 | Payload | ScreenConnect service installation | High | Event Logs / EDR |
| D-012 | Payload | Security-named folder in Documents | Medium | File Monitoring |
| D-013 | Payload | log_PIN_*.txt file creation | High | EDR / DLP |
