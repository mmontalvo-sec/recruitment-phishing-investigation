# Incident Report: Recruitment-Themed Phishing Campaign

**Report Date:** June 8, 2026
**Incident Date:** June 3, 2026
**Status:** Closed
**Severity:** High
**Analyst:** Mathew Montalvo
**Classification:** Public (Sanitized)

---

## Executive Summary

On June 3, 2026, a multi-stage phishing campaign was identified targeting an IT Support Specialist job applicant. Threat actors impersonated a legitimate technology company's recruiting team to deliver a credential-harvesting and remote access payload via a cloned Microsoft Teams interface and a malicious JavaScript downloader.

The campaign demonstrated above-average operational sophistication: realistic recruiter communication, accurate brand impersonation, a convincing fake Microsoft Teams pre-join experience, and a layered payload delivery mechanism using Windows Script Host, Cloudflare R2 hosting, and Telegram-based command-and-control reporting.

The incident was identified, contained, and fully remediated. The endpoint was restored via clean Windows reinstall. All available evidence was preserved for documentation and reporting.

---

## Scope

- **Affected systems:** One personal Windows laptop
- **Affected accounts:** Personal Gmail account (used for job applications)
- **Attack surface:** Email, browser, local filesystem
- **Confirmed data accessed by attacker:** Hostname, username, public IP address, operating system name (collected and transmitted via Telegram by the malicious script upon execution)
- **Suspected but unconfirmed:** PIN value capture (observed in local artifact logs)
- **Not confirmed:** Successful persistent remote access, data exfiltration beyond telemetry

---

## Background

The analyst applied for an IT Support Specialist position advertised through Indeed on or around May 24, 2026. The listing was attributed to Mercer Bucks Technology. The Indeed listing was later found to be unavailable, consistent with a pattern where fraudulent job postings are removed after initial victim contact is established.

---

## Initial Access Vector

**Vector:** Spear-phishing via email
**Sender:** `tammydotson[@]officesdesknote[.]com`
**Impersonated entity:** Mercer Bucks Technology
**Delivery platform:** Gmail (personal)

The attacker initiated contact using a plausible recruiter name ("Tammy Dotson") and a domain (`officesdesknote[.]com`) that did not belong to Mercer Bucks Technology. The email referenced the analyst's genuine job application, suggesting either the attacker scraped Indeed job seeker data or monitored responses to fraudulent postings.

---

## Social Engineering Flow

The campaign followed a structured multi-step social engineering sequence:

**Step 1 — Recruiter Outreach (June 3, 11:09 AM)**
Initial email received thanking the analyst for applying and requesting interview availability for a Microsoft Teams interview.

**Step 2 — Availability Response (June 3, 11:49 AM)**
Analyst replied with Monday, June 8 availability between 9:00 AM and 11:00 AM EST.

**Step 3 — Fake Teams Meeting Invite (June 3, 2:03 PM)**
The attacker sent a follow-up email formatted as a Microsoft Teams meeting invitation. It included:
- A "Join Microsoft Teams Session" button
- A fallback link
- Meeting details: Date June 8, Time 9:00 AM, Organizer listed as "Rebecca Griffins"

The email used Teams-style formatting to create a legitimate appearance. The embedded link, however, directed to a non-Microsoft domain.

**Step 4 — Fake Teams Pre-Join Page**
When the link was opened, it redirected to `ms-teaminvite[.]team/MS-Teams`, which displayed a cloned Microsoft Teams web interface. The page:
- Requested camera and microphone browser permissions
- Displayed a Teams-style meeting name input field
- Showed audio device settings (Windows Microphone, Windows Speakers)
- Presented a "Computer audio - Connected" status indicator
- Displayed a "Join now" button styled to match the legitimate Teams web client

**Step 5 — Fake Update Prompt and Payload Delivery**
The fake page subsequently presented a prompt claiming Microsoft Teams required an update, and triggered the download of a JavaScript file (`MS-TeamSetup.js`). Legitimate Microsoft Teams updates are not delivered as JavaScript files from third-party domains.

**Step 6 — Script Execution**
The downloaded JavaScript file was executed. Windows SmartScreen displayed an "unknown publisher" warning before execution, which was bypassed. The script began its payload delivery routine.

**Step 7 — Anomalous Behavior Detected**
The analyst observed suspicious behavior including unexpected Phone Link / PIN activity and later discovered the `Documents\WinSec` folder containing timestamped PIN-style input logging files.

**Step 8 — Verification Attempt (June 3, 8:54 PM)**
The analyst sent an email to the attacker requesting an official Microsoft Teams calendar invitation or a direct `teams.microsoft.com` link.

**Step 9 — Attacker Confirmation of Malicious Intent (June 4, 2:24 AM)**
The attacker replied: "You need to Proceed and install it." This response, arriving at an unusual hour and refusing to provide a legitimate link, confirmed the workflow was not from a legitimate recruiter.

---

## Technical Findings

### Phishing Domain and Page

The domain `ms-teaminvite[.]team` was registered to impersonate Microsoft Teams collaboration infrastructure. The landing page at `/MS-Teams` was a purpose-built clone of the Microsoft Teams web pre-join experience.

Page source analysis confirmed:
- Teams logo loaded from an external image hosting service (not Microsoft CDN)
- A complete functional mock of the Teams pre-join interface including camera feed, audio device selection, and join controls
- A simulated "Meeting Room" environment with fake meeting timer and participant UI elements
- A device blocking page that claimed the meeting required a Windows laptop, functioning as a pretext to keep victims on the fake page
- JavaScript logic to progress the victim through a fake meeting flow before triggering the payload download

### Malicious Script (MS-TeamSetup.js)

The recovered JavaScript file was a Windows Script Host (WSH) JScript downloader with the following confirmed capabilities:

**Payload Retrieval:**
- Downloaded an MSI file from a Cloudflare R2 hosted URL
- The remote filename referenced ScreenConnect ClientSetup
- The local filename was set to `Microsoft Teams.msi` to appear legitimate
- Used `WinHttp.WinHttpRequest.5.1` and `Msxml2.ServerXMLHTTP.6.0` ActiveX objects for HTTP download
- Wrote the payload to disk using `ADODB.Stream`

**Telemetry Collection:**
- Collected hostname via `%COMPUTERNAME%` environment variable
- Collected username via `%USERNAME%` environment variable
- Collected public IP address by querying `api.ipify.org` and `icanhazip.com`
- Collected operating system name via WMI query (`Win32_OperatingSystem`)

**Command and Control (C2):**
- Used a hardcoded Telegram bot token and chat ID to report execution status
- Sent execution alerts at three stages: script start, download complete, and install result
- Alerts included all collected telemetry (hostname, user, IP, OS) and MSI install exit code
- Telegram token and chat ID are redacted in all public materials

**Execution:**
- Invoked `msiexec.exe /i` to silently install the downloaded MSI
- Displayed a fake "Microsoft Teams is preparing your installation" popup during execution to maintain the illusion of a legitimate installer

**ActiveX Objects Used:**
- `WScript.Shell`
- `Scripting.FileSystemObject`
- `WinHttp.WinHttpRequest.5.1`
- `Msxml2.ServerXMLHTTP.6.0`
- `ADODB.Stream`

### Local Endpoint Artifacts

**WinSec Folder:**
A folder was observed at `Documents\WinSec` containing two text files:
- `log_PIN_2026-06-03_17-59-35.txt`
- `log_PIN_2026-06-03_18-00-23.txt`

The first log file contained:
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

This artifact is consistent with a PIN-style keystroke logger. The log captured incremental keystroke input and the final submitted value. The file naming convention and folder name suggest the dropped payload included a credential capture component beyond the ScreenConnect remote access installer.

**Assessment:** This artifact strongly suggests the MSI payload included or dropped a credential harvesting component targeting Windows Hello PIN authentication. This is classified as a strong suspicion, not confirmed credential theft, pending further analysis of the MSI payload itself.

**ScreenConnect DLL Remnant:**
After the analyst attempted file removal, a ScreenConnect-related DLL was identified remaining under `Program Files (x86)`, consistent with a partial ScreenConnect/ConnectWise installation that was not fully uninstalled.

---

## Timeline

See [`TIMELINE.md`](TIMELINE.md) for the full chronological event log.

---

## Indicators of Compromise

See [`IOCS.md`](IOCS.md) for the full defanged IOC list.

---

## Containment Actions

1. Stopped all interaction with the phishing domain and attacker email
2. Closed and did not reopen the phishing URL
3. Isolated the affected laptop from network access
4. Preserved all available evidence: email exchange, screenshots, script text, local artifact screenshots
5. Removed the `Documents\WinSec` folder and associated log files
6. Attempted manual removal of ScreenConnect-related files
7. Identified a remaining ScreenConnect DLL under `Program Files (x86)`
8. Ran a full Microsoft Defender antivirus scan

---

## Remediation Actions

1. Full Microsoft Defender scan completed
2. Clean Windows reinstall performed to eliminate any persistence mechanisms not identified through manual review
3. Post-reinstall: verified no ScreenConnect or ConnectWise processes or services present
4. Endpoint returned to trusted state

**Rationale for clean reinstall:** Given the script's capability to install ScreenConnect (a remote access tool), the presence of PIN-style logging artifacts, and uncertainty about full payload execution scope, a clean reinstall was the appropriate remediation decision to restore full endpoint trust.

---

## Limitations of This Investigation

- The MSI payload (`SscreenConnect.ClientSetup (6).msi`) was not recovered for dynamic or sandbox analysis
- The Telegram C2 channel was not monitored; attacker-side telemetry was not captured
- Full execution scope of the MSI payload is unknown
- Whether data was exfiltrated beyond the initial Telegram telemetry alert is not confirmed
- Whether the legitimate Mercer Bucks Technology company was compromised (vs. simply impersonated) is not confirmed

---

## Conclusion

This incident represents a well-constructed recruitment phishing campaign targeting IT job seekers. The attack chain combined social engineering, brand impersonation, UI cloning, scripted payload delivery, remote access tooling, and local credential capture behavior into a cohesive multi-stage operation.

Early red flags were present, including the non-Microsoft sender domain, the non-Microsoft meeting URL, and the JavaScript file format of the supposed Teams installer. The inability to obtain a legitimate calendar invitation from the recruiter served as the final confirmation of malicious intent.

The incident was contained, documented, and remediated. All findings have been preserved for awareness and reporting purposes.

Detection, documentation, and proper remediation are as important as prevention. This case study demonstrates applied skills in phishing identification, static analysis, IOC extraction, endpoint containment, and incident reporting.
