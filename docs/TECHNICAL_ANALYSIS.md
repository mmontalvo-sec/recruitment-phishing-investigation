# Technical Analysis

**Case:** RPI-2026-001
**Artifact:** MS-TeamSetup.js (recovered JavaScript dropper)
**Analysis Type:** Static / Manual Code Review
**Analyst:** Mathew Montalvo
**Date:** June 2026

---

## Overview

The recovered artifact is a Windows Script Host (WSH) JScript file designed to masquerade as a Microsoft Teams installer. Static analysis of the script reveals a structured dropper with four primary functions: telemetry collection, command-and-control reporting via Telegram, payload download from Cloudflare R2 cloud storage, and execution of the downloaded MSI via `msiexec.exe`.

No sandbox or dynamic analysis was performed. All findings are derived from source code review.

---

## Execution Environment

| Property | Value |
|---|---|
| Runtime | Windows Script Host (WSH) via `cscript.exe` or `wscript.exe` |
| Language | JScript (Microsoft's WSH JavaScript implementation) |
| Execution trigger | User double-click or shell execution of `.js` file |
| Compatibility | Windows XP and later (WSH is built into Windows) |
| Execution warning | Windows SmartScreen "unknown publisher" warning displayed prior to execution |

The script uses WSH's native `ActiveXObject` interface extensively, which provides direct access to Windows COM automation without requiring any additional dependencies or elevated privileges for the initial download and execution stages.

---

## Script Structure

The entire script is wrapped in an immediately-invoked function expression (IIFE), a common technique to scope variables and reduce the risk of global namespace collisions. This is a characteristic shared with some commodity malware families and legitimate JavaScript practice alike.

```
(function() {
    // Configuration variables
    // Helper functions
    // Main execution flow
})();
```

### Configuration Block (Hardcoded)

The script contains four hardcoded configuration values at the top:

| Variable | Value | Purpose |
|---|---|---|
| `MSI_URL` | `hxxps://pub-[REDACTED].r2.dev/SscreenConnect.ClientSetup%20(6).msi` | Remote payload location |
| `MSI_FILE_NAME` | `Microsoft Teams.msi` | Local disguised filename |
| `TELEGRAM_BOT_TOKEN` | `[REDACTED]` | Telegram bot authentication |
| `TELEGRAM_CHAT_ID` | `[REDACTED]` | Attacker-controlled Telegram channel |

The choice to hardcode all configuration inline (rather than fetching a config from a remote server) suggests a relatively simple operational setup. No obfuscation, encryption, or dynamic decoding was applied to the configuration values.

---

## Functional Breakdown

### 1. Victim Identification and Telemetry Collection

The script collects four data points from the victim system before and after payload delivery:

**Hostname:**
```javascript
shell.ExpandEnvironmentStrings("%COMPUTERNAME%")
```

**Username:**
```javascript
shell.ExpandEnvironmentStrings("%USERNAME%")
```

**Public IP Address:**
The script queries two external services sequentially, using the first successful response:
- `hxxps://api[.]ipify[.]org`
- `hxxps://icanhazip[.]com`

**Operating System:**
WMI query using `GetObject("winmgmts:\\.\root\cimv2")` with `SELECT Caption FROM Win32_OperatingSystem`.

This telemetry profile allows the attacker to identify the specific victim and their network location immediately upon script execution, before payload installation completes.

---

### 2. Telegram C2 Reporting

The script reports execution progress to an attacker-controlled Telegram bot at three checkpoints:

| Checkpoint | Trigger | Data Sent |
|---|---|---|
| Start | Script begins | Hostname, user, IP, OS, current timestamp |
| Download Complete | MSI file written to disk | Hostname, user, IP, OS, file path |
| Install Result | `msiexec.exe` exits | Hostname, user, IP, OS, install status, exit code |

This gives the attacker real-time visibility into which victims executed the script, whether the payload downloaded successfully, and whether installation completed or failed.

The Telegram API call uses HTTP GET with URL-encoded parameters, sent via `WinHttp.WinHttpRequest.5.1`. TLS 1.2 is explicitly enabled. Two retry attempts are made per report with a 500ms delay between them.

**Significance:** The Telegram C2 mechanism means the attacker does not need to operate any custom infrastructure for victim tracking. The bot API is free, globally available, and encrypted. This is a known technique in commodity and semi-targeted malware.

---

### 3. Payload Download

The MSI payload is downloaded using a two-stage fallback mechanism:

**Primary:** `WinHttp.WinHttpRequest.5.1`
**Fallback:** `Msxml2.ServerXMLHTTP.6.0`

If the primary HTTP client fails (for example, due to WinHTTP proxy settings), the script automatically retries using MSXML. This increases reliability across different victim environments.

The downloaded binary response body is written to disk using `ADODB.Stream` with `Type = 1` (binary), targeting `%TEMP%\Microsoft Teams.msi`.

The script includes a popup dialog during download:
> "Microsoft Teams is preparing your installation. Click OK to continue."

This dialog serves to maintain the victim's belief that a legitimate Teams installer is running and to occupy user attention during the download phase.

---

### 4. Payload Execution

After confirming the file exists on disk, the script executes:
```
msiexec.exe /i "%TEMP%\Microsoft Teams.msi"
```

Using `shell.Run()` with `bWaitOnReturn = true`, meaning the script waits for `msiexec.exe` to complete before sending the final Telegram report.

Exit codes are interpreted:

| Code | Meaning |
|---|---|
| 0 | Success |
| 3010 | Success, reboot required |
| 1641 | Success, reboot initiated |
| 1638 | Already installed |
| 1602 | Canceled by user |
| Other | Failure with exit code |

**Actual payload:** The MSI remote filename (`SscreenConnect.ClientSetup (6).msi`) confirms the final payload is a ScreenConnect (ConnectWise Control) remote access client. ScreenConnect is a legitimate commercial remote desktop tool. When installed by a threat actor, it provides persistent, authenticated remote access to the victim system without the victim's knowledge.

---

## Phishing Page Infrastructure

### Domain
`ms-teaminvite[.]team` was registered specifically to impersonate Microsoft Teams meeting infrastructure. The domain uses a plausible keyword structure (`ms` + `team` + `invite`) that could appear legitimate to a victim in a hurry.

### Page Design
Source code review of the fake page confirmed:
- Teams logo loaded from external image hosting (not Microsoft CDN)
- Complete functional clone of Teams web pre-join experience
- Camera feed simulation with live browser camera access request
- Audio device selection displaying real Windows device names
- A secondary "device blocking" page claiming the meeting required a Windows PC (pretext to prevent mobile users from noticing the mismatch)
- A meeting room simulation page suggesting the victim had entered an active meeting

### Update Mechanism
The page used JavaScript to transition between phases of the fake experience. After a simulated loading period, the page displayed the fake update requirement and triggered the `.js` file download. This mirrors a technique used by various web-based malware delivery campaigns that imitate browser or application update flows.

---

## PIN Logging Artifact

Two log files were observed in `Documents\WinSec\`:

```
Windows Security Log - 06/03/2026 17:59:35
Authentication Type: PIN
================================
06/03/2026 18:00:06 - Input: 2
06/03/2026 18:00:07 - Input: 26
06/03/2026 18:00:07 - Input: 264
06/03/2026 18:00:07 - Input: 2642
06/03/2026 18:00:08 - SUBMITTED: 2642
```

**Assessment:**
- The log format mimics a Windows Security Log header to appear legitimate if casually observed
- The content captures incremental keystrokes (building the PIN digit by digit) and the final submitted value
- The folder name (`WinSec`) and log naming convention suggest a deliberate attempt to blend with legitimate Windows security logging
- The timestamps place this activity approximately 1 hour and 40 minutes after the fake Teams meeting invite was received, and within the same session as the script execution
- This behavior is consistent with a keylogger or credential capture module dropped by the ScreenConnect MSI payload or a co-delivered component
- The exact mechanism of capture (Windows Hello hook, custom input monitor, etc.) could not be confirmed without dynamic analysis of the MSI

**Classification:** Strong suspicion of PIN credential capture. Not confirmed pending MSI payload analysis.

---

## Severity Assessment

| Factor | Assessment |
|---|---|
| Payload capability | Remote access (ScreenConnect) + probable credential capture |
| C2 reliability | High (Telegram API, no custom infrastructure required) |
| Operational sophistication | Medium-High (convincing social engineering, accurate UI clone, layered payload) |
| Attribution | Not attempted |
| Data confirmed exfiltrated | Hostname, username, public IP, OS (via Telegram at execution) |
| Data suspected exfiltrated | Windows Hello PIN value |
| Persistence | Likely (ScreenConnect installs as a service); not confirmed post-reinstall |

---

## Analyst Notes

The recovered script contains no obfuscation. All logic is readable plaintext JScript. This suggests either the attacker did not prioritize evasion at the script layer (relying on the social engineering context to lower suspicion), or the script was derived from a commodity template not specifically hardened against static analysis.

The double typo in the remote MSI filename (`SscreenConnect` with two capital S characters and the `(6)` suffix) is consistent with a threat actor managing multiple campaign instances, where the numbering reflects different deployment batches or victim groups.
