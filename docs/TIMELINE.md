# Incident Timeline

**Incident ID:** RPI-2026-001
**Timezone:** Eastern Time (ET)

---

| Date | Time | Event | Category |
|---|---|---|---|
| May 24, 2026 | Unknown | Analyst applied for IT Support Specialist position through Indeed, attributed to Mercer Bucks Technology | Pre-incident |
| June 3, 2026 | 11:09 AM | Initial phishing email received from `tammydotson[@]officesdesknote[.]com`, referencing the Indeed application and requesting interview availability | Initial Contact |
| June 3, 2026 | 11:49 AM | Analyst replied to recruiter email with Monday June 8 availability, 9:00 AM to 11:00 AM EST | Victim Response |
| June 3, 2026 | 2:03 PM | Attacker sent a second email formatted as a Microsoft Teams meeting invitation for June 8 at 9:00 AM, including a fake join link | Payload Staging |
| June 3, 2026 | Afternoon | Meeting link opened in Brave browser; browser navigated to `ms-teaminvite[.]team/MS-Teams` | Payload Delivery - Stage 1 |
| June 3, 2026 | Afternoon | Fake Teams pre-join page loaded; site requested camera and microphone permissions via browser prompt | Payload Delivery - Stage 2 |
| June 3, 2026 | Afternoon | Fake page presented a Teams update requirement; JavaScript file (`MS-TeamSetup.js` or similar) downloaded | Payload Delivery - Stage 3 |
| June 3, 2026 | Afternoon | JavaScript file executed; Windows SmartScreen unknown publisher warning was bypassed; script began execution | Execution |
| June 3, 2026 | ~5:59 PM | `Documents\WinSec` folder created on local filesystem; `log_PIN_2026-06-03_17-59-35.txt` written (per file timestamp) | Artifact - Credential Capture |
| June 3, 2026 | ~6:00 PM | `log_PIN_2026-06-03_18-00-23.txt` written; log captured incremental PIN input and final submitted value | Artifact - Credential Capture |
| June 3, 2026 | 8:54 PM | Analyst sent email to attacker requesting an official Microsoft Teams calendar invite or a direct `teams.microsoft.com` link | Detection / Verification |
| June 4, 2026 | 2:24 AM | Attacker replied: "You need to Proceed and install it." No legitimate link was provided. | Attacker Confirmation |
| June 5, 2026 | 1:21 AM | Analyst revisited the domain; Cloudflare displayed a "Suspected Phishing" warning for `ms-teaminvite[.]team/MS-Teams` | External Validation |
| June 5, 2026 | 1:21 AM | Screenshots captured of fake Teams page, Cloudflare phishing warning, browser URL bar, WinSec artifacts, and page source | Evidence Preservation |
| Post June 5 | — | Endpoint isolated; manual file removal attempted; ScreenConnect DLL remnant identified in `Program Files (x86)` | Containment |
| Post June 5 | — | Full Microsoft Defender scan run | Containment |
| Post June 5 | — | Clean Windows reinstall performed | Remediation |
| Post June 5 | — | Endpoint verified clean; evidence packaged and sanitized for documentation | Closure |
| June 8, 2026 | — | Investigation documented and published as public portfolio case study | Reporting |

---

## Timeline Notes

- The script's Telegram bot reported execution status in real time, meaning the attacker received hostname, username, public IP, and OS details at the moment of script execution.
- The WinSec PIN log timestamps (17:59:35 and 18:00:23) suggest the credential capture component activated shortly after the script ran, consistent with same-session activity.
- The attacker's 2:24 AM response reinforces the operational pattern of automated or offshore-operated campaigns that run outside standard business hours.
- The Cloudflare phishing flag on June 5 indicates the domain had already been reported by others, suggesting this campaign targeted multiple victims.
