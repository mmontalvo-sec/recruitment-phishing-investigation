# MITRE ATT&CK Mapping

**Case:** RPI-2026-001
**Framework Version:** ATT&CK v15
**Assessment Type:** Static analysis and behavioral observation
**Analyst:** Mathew Montalvo

Confidence levels reflect the quality of supporting evidence available:
- **Confirmed:** Directly observed in artifact or evidence
- **High:** Strong behavioral or circumstantial evidence
- **Medium:** Consistent with observed behavior; not directly confirmed
- **Low:** Plausible based on payload capabilities; not directly observed

---

## Tactic: Initial Access

### T1566.002 — Phishing: Spearphishing Link

**Confidence:** Confirmed

The attacker sent a targeted email to a specific individual who had applied for a job posting. The email contained a hyperlink disguised as a Microsoft Teams meeting join button. The link redirected to a threat-actor-controlled domain. The use of a genuine job application context to personalize the lure elevates this from generic phishing to spearphishing.

**Evidence:** Email exchange PDF, browser screenshots showing domain mismatch.

---

## Tactic: Execution

### T1059.007 — Command and Scripting Interpreter: JavaScript

**Confidence:** Confirmed

The primary dropper was a JScript file executed via Windows Script Host (`wscript.exe` or `cscript.exe`). WSH JScript is a native Windows scripting environment that requires no additional software installation. The script used ActiveX COM automation objects to perform all download and execution operations.

**Evidence:** Recovered script artifact (`MS-TeamSetup.js`).

---

### T1204.002 — User Execution: Malicious File

**Confidence:** Confirmed

The victim was socially engineered into downloading and manually executing the JavaScript dropper. The script relied entirely on user action to run. Windows SmartScreen displayed an "unknown publisher" warning, which was bypassed by the user under the belief the file was a legitimate Teams update.

**Evidence:** Script content, social engineering context, attacker instruction to "proceed and install."

---

## Tactic: Defense Evasion

### T1036.005 — Masquerading: Match Legitimate Name or Location

**Confidence:** Confirmed

The malicious JavaScript file was named to resemble a legitimate Microsoft Teams installer (`MS-TeamSetup.js`, `MSTeamsSetup.js`). The downloaded MSI payload was saved locally as `Microsoft Teams.msi` despite its remote filename identifying it as a ScreenConnect installer. A fake popup dialog displaying "Microsoft Teams is preparing your installation" reinforced the masquerade during execution.

**Evidence:** Script variable assignments for `MSI_FILE_NAME` and popup text content.

---

### T1027 — Obfuscated Files or Information

**Confidence:** Low

The script itself was not obfuscated. However, the payload was disguised at both the filename level and through a social engineering wrapper. Low confidence on technical obfuscation; the masquerading technique above is the primary evasion mechanism.

---

## Tactic: Discovery

### T1082 — System Information Discovery

**Confidence:** Confirmed

The script queried the operating system name via WMI (`SELECT Caption FROM Win32_OperatingSystem`) and collected hostname via the `%COMPUTERNAME%` environment variable. This information was transmitted to the attacker via Telegram.

**Evidence:** Script source code, `getOsName()` function and `formatExecutionAlert()` telemetry fields.

---

### T1033 — System Owner/User Discovery

**Confidence:** Confirmed

The script collected the current username via `%USERNAME%` environment variable and included it in Telegram execution reports. Combined with hostname, this allows the attacker to identify the specific user on the compromised system.

**Evidence:** Script source code, `formatExecutionAlert()` telemetry fields.

---

### T1016 — System Network Configuration Discovery

**Confidence:** Confirmed

The script queried two external services (`api.ipify.org`, `icanhazip.com`) to determine the victim's public IP address, which was included in Telegram telemetry. This provides the attacker with geographic and network context about the victim.

**Evidence:** Script source code, `getPublicIp()` function.

---

## Tactic: Command and Control

### T1102.002 — Web Service: Bidirectional Communication (Telegram)

**Confidence:** Confirmed

The script used the Telegram Bot API as an outbound-only command and control channel. Execution status, victim telemetry, and install results were transmitted to an attacker-controlled Telegram bot. Telegram was chosen likely because it is a legitimate service that bypasses domain reputation blocks, uses encrypted HTTPS, and requires no attacker-operated infrastructure.

**Evidence:** Script source code, `sendTelegram()` function, hardcoded bot token and chat ID.

---

## Tactic: Lateral Movement / Persistence

### T1219 — Remote Access Software

**Confidence:** High

The MSI payload was identified as a ScreenConnect (ConnectWise Control) client installer. ScreenConnect, when installed by a threat actor, provides persistent remote access to the victim machine including file transfer, shell access, and screen viewing. The ScreenConnect client typically installs as a Windows service, enabling persistence across reboots.

**Evidence:** Remote MSI filename in script (`SscreenConnect.ClientSetup (6).msi`), ScreenConnect DLL remnant found in `Program Files (x86)` post-incident.

---

## Tactic: Collection

### T1056.001 — Input Capture: Keylogging

**Confidence:** Medium

Log files in `Documents\WinSec\` captured incremental PIN keystrokes and a final submitted value in a format consistent with a keylogger or Windows Hello input monitor. The exact capture mechanism was not confirmed through dynamic analysis, but the artifact behavior matches input capture functionality.

**Evidence:** WinSec folder screenshots, log file content showing keystroke-by-keystroke PIN capture.

---

### T1056.004 — Input Capture: Credential API Hooking

**Confidence:** Low

PIN capture behavior is consistent with hooking Windows credential input APIs, but this was not confirmed. Dynamic analysis of the MSI payload would be required to assess this technique with higher confidence.

---

## Tactic: Exfiltration

### T1041 — Exfiltration Over C2 Channel

**Confidence:** High

System telemetry (hostname, username, public IP, OS name) was transmitted to the attacker via the Telegram C2 channel upon script execution and again upon payload download. This constitutes confirmed exfiltration of host reconnaissance data. Potential PIN exfiltration is assessed at medium confidence pending artifact analysis.

**Evidence:** Script source code, Telegram alert formatting function content.

---

## Summary Table

| Technique ID | Technique Name | Tactic | Confidence |
|---|---|---|---|
| T1566.002 | Spearphishing Link | Initial Access | Confirmed |
| T1059.007 | JavaScript | Execution | Confirmed |
| T1204.002 | Malicious File | Execution | Confirmed |
| T1036.005 | Match Legitimate Name or Location | Defense Evasion | Confirmed |
| T1027 | Obfuscated Files or Information | Defense Evasion | Low |
| T1082 | System Information Discovery | Discovery | Confirmed |
| T1033 | System Owner/User Discovery | Discovery | Confirmed |
| T1016 | System Network Configuration Discovery | Discovery | Confirmed |
| T1102.002 | Web Service: Bidirectional Communication | Command and Control | Confirmed |
| T1219 | Remote Access Software | Command and Control | High |
| T1056.001 | Keylogging | Collection | Medium |
| T1056.004 | Credential API Hooking | Collection | Low |
| T1041 | Exfiltration Over C2 Channel | Exfiltration | High |
