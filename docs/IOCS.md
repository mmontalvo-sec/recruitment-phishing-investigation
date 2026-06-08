# Indicators of Compromise (IOCs)

**Case:** RPI-2026-001
**Classification:** Public (Sanitized)
**Last Updated:** June 8, 2026

All domains are defanged using bracket notation to prevent accidental clicks or automated resolution. Do not remove brackets before using in production threat intelligence platforms.

---

## Network Indicators

### Domains

| Indicator | Type | Context | Confidence |
|---|---|---|---|
| `ms-teaminvite[.]team` | Phishing Domain | Hosted fake Microsoft Teams pre-join page | Confirmed |
| `officesdesknote[.]com` | Sender Domain | Attacker email domain impersonating recruiter | Confirmed |
| `api[.]telegram[.]org` | C2 Communication | Used by script to send execution alerts via Telegram bot API | Confirmed (from script) |
| `api[.]ipify[.]org` | Victim Recon | Used by script to retrieve target public IP address | Confirmed (from script) |
| `icanhazip[.]com` | Victim Recon | Backup IP lookup service used by script | Confirmed (from script) |
| `pub-[REDACTED][.]r2[.]dev` | Payload Host | Cloudflare R2 bucket hosting the ScreenConnect MSI payload | Confirmed (URL redacted) |

### URLs

| Indicator | Type | Context |
|---|---|---|
| `ms-teaminvite[.]team/MS-Teams` | Phishing URL | Entry point for fake Teams experience and payload delivery |
| `hxxps://pub-[REDACTED][.]r2[.]dev/SscreenConnect[.]ClientSetup%20(6)[.]msi` | Payload URL | Direct download link for ScreenConnect MSI (defanged, host redacted) |

---

## Email Indicators

| Indicator | Type | Context | Confidence |
|---|---|---|---|
| `tammydotson[@]officesdesknote[.]com` | Sender Address | Initial phishing contact impersonating Mercer Bucks Technology recruiting | Confirmed |
| `officesdesknote[.]com` | Sender Domain | Does not belong to Mercer Bucks Technology | Confirmed |
| "Tammy Dotson" | Display Name | Fake recruiter persona | Confirmed |
| "Rebecca Griffins" | Meeting Organizer Name | Fake Teams meeting organizer name in invite | Confirmed |
| "Mercer Bucks Technology" | Impersonated Entity | Legitimate company impersonated in the campaign | Confirmed |
| `Your Application to Mercer Bucks Technology - support Specialist Position` | Email Subject | Phishing email subject line | Confirmed |

---

## File Indicators

| Indicator | Type | Context | Confidence |
|---|---|---|---|
| `MS-TeamSetup.js` | Malicious File | JavaScript dropper downloaded from phishing site | Confirmed |
| `MSTeamsSetup.js` | Malicious File (alt name) | Alternate filename variant observed | Probable |
| `Microsoft Teams.msi` | Dropped File (local name) | ScreenConnect MSI saved under disguised local filename | Confirmed (from script) |
| `SscreenConnect.ClientSetup (6).msi` | Dropped File (remote name) | Actual remote filename of ScreenConnect MSI on Cloudflare R2 | Confirmed (from script) |

---

## Filesystem Artifacts

| Indicator | Type | Context | Confidence |
|---|---|---|---|
| `%USERPROFILE%\Documents\WinSec\` | Suspicious Folder | Created by payload; contained PIN-style keystroke logs | Confirmed |
| `log_PIN_YYYY-MM-DD_HH-MM-SS.txt` | Log File Pattern | Naming pattern of PIN capture log files in WinSec folder | Confirmed |
| `log_PIN_2026-06-03_17-59-35.txt` | Log File | First observed PIN log file | Confirmed |
| `log_PIN_2026-06-03_18-00-23.txt` | Log File | Second observed PIN log file | Confirmed |
| `%TEMP%\Microsoft Teams.msi` | Dropped File Path | MSI dropped to TEMP directory during script execution | Confirmed (from script) |
| ScreenConnect-related DLL | Persistence Remnant | Identified in `Program Files (x86)` after manual removal attempt | Confirmed |

---

## Malware / Script Indicators

| Indicator | Type | Context | Confidence |
|---|---|---|---|
| `WScript.Shell` | ActiveX Object | Used for shell command execution and environment variable expansion | Confirmed |
| `Scripting.FileSystemObject` | ActiveX Object | Used for file existence checks and deletion | Confirmed |
| `WinHttp.WinHttpRequest.5.1` | ActiveX Object | Used for HTTP download of MSI payload and Telegram C2 | Confirmed |
| `Msxml2.ServerXMLHTTP.6.0` | ActiveX Object | Fallback HTTP client for payload download | Confirmed |
| `ADODB.Stream` | ActiveX Object | Used to write downloaded binary MSI to disk | Confirmed |
| `msiexec.exe /i` | Execution Method | Used to silently install the downloaded MSI | Confirmed |
| Telegram Bot API | C2 Mechanism | Execution alerting; bot token and chat ID redacted | Confirmed |
| `[REDACTED_TOKEN]` | Telegram Bot Token | Hardcoded in script; redacted for public release | Confirmed (redacted) |
| `[REDACTED_CHAT_ID]` | Telegram Chat ID | Hardcoded in script; redacted for public release | Confirmed (redacted) |

---

## Threat Intelligence Notes

- The Cloudflare R2 payload hosting is consistent with attackers abusing legitimate cloud infrastructure to evade domain-based reputation blocking.
- The Telegram C2 mechanism is a known technique used by commodity and mid-tier threat actors for real-time victim tracking without operating their own C2 infrastructure.
- The ScreenConnect (ConnectWise Control) payload is a legitimate commercial remote access tool frequently abused in tech support scams and targeted intrusion campaigns.
- The WinSec PIN logging behavior, if confirmed as a payload component, suggests an additional credential theft objective beyond remote access establishment.
- The double typo in the remote MSI filename (`SscreenConnect` with double S, filename ending with `(6)`) is consistent with a commodity or semi-automated campaign reusing infrastructure across multiple victims.

---

## Redaction Notes

The following values are present in the recovered script but have been intentionally removed from this public document:

- Telegram bot token (full value)
- Telegram chat ID (full value)
- Cloudflare R2 bucket subdomain (partial redaction applied to URL)
- Captured PIN value (observed in log artifact)
