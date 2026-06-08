# Evidence Index

**Case:** RPI-2026-001
**Classification:** Public (Sanitized)
**Last Updated:** June 8, 2026

This folder contains evidence preserved during the incident investigation. All artifacts have been reviewed and sanitized for public release. Sensitive values have been redacted. Raw malicious files are not included in this public repository.

---

## Evidence Inventory

| # | Artifact | Type | Description | Status |
|---|---|---|---|---|
| 01 | `Email_Exchange` | Email (PDF export) | Full email chain between analyst and attacker, from initial recruiter contact through the "proceed and install" response | Preserved — personal email address redacted |
| 02 | `phishing_teams_web_landing_page` | Screenshot | Browser showing `ms-teaminvite.team/MS-Teams` loading with the Teams logo — domain visible in address bar | Preserved |
| 03 | `web_teams_ui` | Screenshot | Fake Teams pre-join page with camera/mic permission request from `ms-teaminvite.team` (browser prompt visible) | Preserved |
| 04 | `web_teams_ui_2` | Screenshot | Fake Teams pre-join page with camera access denied state, audio device selection visible | Preserved |
| 05 | `TEAMS_invite_forward_new_warning` | Screenshot | Cloudflare "Suspected Phishing" warning displayed when revisiting `ms-teaminvite.team/MS-Teams` on June 5, 2026 | Preserved |
| 06 | `security_pin_logs` | Screenshot | `Documents\WinSec` folder contents showing two `log_PIN_*.txt` files and the content of the first log with PIN-style keystroke capture | Preserved — submitted PIN value visible in log; included as evidence of artifact, not as credential exposure |
| 07 | `msteams_setup_js_txt` | Text File | Source code of the malicious JavaScript dropper recovered from the phishing site | Preserved — Telegram bot token, Telegram chat ID, and Cloudflare R2 URL partially redacted in public version |
| 08 | `msteams_inspect_element` | PDF | Browser page source of the fake Teams page (`view-source:ms-teaminvite.team/MS-Teams`) captured via inspect element | Preserved |
| 09 | `msteams_after_verification` | PDF | Page state after Cloudflare verification challenge at the phishing domain | Preserved |

---

## Redaction Notes

The following values have been intentionally redacted or omitted across all public materials:

| Redacted Item | Location | Reason |
|---|---|---|
| Telegram Bot Token | Script (msteams_setup_js_txt) | Active credential; could be used to interact with attacker's bot |
| Telegram Chat ID | Script (msteams_setup_js_txt) | Active C2 identifier |
| Full Cloudflare R2 URL | Script, IOCs doc | Payload URL may still serve malware |
| Submitted PIN Value | security_pin_logs screenshot | Personal authentication credential |
| Analyst personal email address | Email exchange | Privacy |

---

## What Is Not Included

- The raw MSI payload (`Microsoft Teams.msi` / `SscreenConnect.ClientSetup (6).msi`) — was not recovered for static analysis and would not be safe to distribute publicly
- Any ScreenConnect configuration data or session keys
- Full unredacted Telegram credentials
- Full unredacted Cloudflare R2 payload URL in clickable form

---

## Chain of Custody Notes

All evidence was collected by the analyst directly from the affected system before remediation. Screenshots were taken using the device camera (photographing the screen) due to the isolated state of the endpoint. The JavaScript source was preserved as a plain text file prior to system wipe. Evidence was packaged and transferred to a clean system for documentation and analysis.

No third-party forensics tools were used. Analysis was performed through manual code review, screenshot comparison, and behavioral observation.
