# Recruitment Phishing Investigation

**Status:** Closed | **Severity:** High | **Endpoint Status:** Remediated

A documented investigation of a sophisticated recruitment-themed phishing campaign targeting IT job applicants. This repository contains sanitized evidence, technical analysis, indicators of compromise (IOCs), and a full incident report suitable for portfolio review.

---

## Overview

In June 2026, I encountered a multi-stage phishing campaign that impersonated a legitimate technology company's recruiting workflow. The threat actors used convincing social engineering, a cloned Microsoft Teams pre-join interface, and a malicious JavaScript downloader to attempt remote access tool deployment on the target endpoint.

This repository documents the investigation, containment, and remediation process from initial detection to clean endpoint recovery.

---

## Why This Matters

Job seekers are high-value targets. The combination of emotional urgency, interview excitement, and brand impersonation makes recruitment phishing particularly effective. This campaign demonstrated:

- Realistic recruiter email workflows with natural response timing
- Pixel-accurate Microsoft Teams UI cloning on a lookalike domain
- A JavaScript-based dropper disguised as a Teams update
- Remote access software (ScreenConnect) delivered as the final payload
- Telegram bot-based execution reporting for real-time attacker awareness
- Local PIN-style input logging behavior suggesting credential collection

This was not a generic phishing attempt. It was operationally targeted and technically layered.

---

## Skills Demonstrated

| Category | Skills Applied |
|---|---|
| Threat Recognition | Social engineering identification, domain spoofing detection, UI cloning analysis |
| Email Analysis | Header review, sender domain validation, recruitment workflow pattern matching |
| Static Analysis | JavaScript source review, ActiveX object identification, payload behavior mapping |
| IOC Extraction | Domain, file name, path, Telegram C2, and ScreenConnect reference extraction |
| Endpoint Forensics | Artifact discovery, suspicious folder/log analysis, persistence indicator review |
| Containment | Isolation, file removal, ScreenConnect DLL remnant identification |
| Remediation | Full Defender scan, clean Windows reinstall, endpoint trust restoration |
| Documentation | Incident report, timeline, MITRE ATT&CK mapping, IOC list, lessons learned |
| Evidence Handling | Sanitized preservation, redaction of sensitive values, public-safe packaging |

---

## Repository Structure

```
recruitment-phishing-investigation/
├── README.md                          ← You are here
├── docs/
│   ├── INCIDENT_REPORT.md             ← Full formal incident report
│   ├── TIMELINE.md                    ← Chronological event timeline
│   ├── IOCS.md                        ← Defanged indicators of compromise
│   ├── TECHNICAL_ANALYSIS.md          ← Script behavior and payload analysis
│   ├── MITRE_ATTACK.md                ← ATT&CK technique mapping with confidence levels
│   ├── LESSONS_LEARNED.md             ← Defensive takeaways and detection opportunities
│   ├── DETECTION_OPPORTUNITIES.md     ← Where controls could have flagged this earlier
│   ├── INTERVIEW_STORY.md             ← Structured interview narrative
│   └── REPORT_TO_COMPANY.md           ← Template for notifying the impersonated company
└── evidence_redacted/
    └── screenshots of evidence 
```

---

## Key Confirmed Findings

- **Initial access vector:** Spear-phishing email from `tammydotson[@]officesdesknote[.]com` impersonating Mercer Bucks Technology recruiting staff
- **Phishing domain:** `ms-teaminvite[.]team/MS-Teams` (not Microsoft-owned)
- **Payload delivery:** JavaScript file (`MS-TeamSetup.js`) downloaded via fake Teams update prompt
- **Script behavior:** Downloaded a ScreenConnect MSI from Cloudflare R2 storage, disguised as `Microsoft Teams.msi`
- **C2 mechanism:** Telegram bot used to report execution status including hostname, username, public IP, and OS
- **Local artifact:** `Documents\WinSec` folder containing `log_PIN_YYYY-MM-DD_HH-MM-SS.txt` files with PIN-style keystroke capture
- **Domain validation:** `ms-teaminvite[.]team` later flagged by Cloudflare as Suspected Phishing

---

## Containment and Remediation Summary

1. Stopped interaction with the phishing domain
2. Preserved all available evidence (screenshots, script, emails, local artifacts)
3. Isolated the affected endpoint from the network
4. Removed suspicious files and the `WinSec` artifact folder
5. Searched for and identified a ScreenConnect-related DLL remnant under `Program Files (x86)`
6. Ran a full Microsoft Defender scan
7. Performed a clean Windows reinstall to restore full endpoint trust
8. Documented all findings for reporting and portfolio use

---

## Evidence

See [`evidence_redacted/README.md`](evidence_redacted)

**Sensitive values intentionally redacted in all public materials:**
- Telegram bot token and chat ID
- Full Cloudflare R2 payload URL
- Captured PIN value
- Personal email addresses

---

## Reported To

- Mercer Bucks Technology (impersonated company) — notification template in `docs/REPORT_TO_COMPANY.md`
- Cloudflare phishing report (domain was flagged and confirmed)

---

## Author

**Mathew Montalvo**
IT Support | NOC | SOC Analyst I Candidate
CompTIA A+ | Network+ | ITF+ | Security+ (In Progress)
B.S. Computer and Network Technology, Minor: Network Security

[GitHub Portfolio](https://mmontalvo-sec.github.io) | [Tech West PC Repair](https://techwestpcr.com)
