# Lessons Learned

**Case:** RPI-2026-001
**Date:** June 2026

---

## What Went Well

**Recognition of red flags under pressure.** Even in a high-stakes job search context where urgency and hope are working against clear thinking, multiple red flags were noticed. The domain mismatch, the JavaScript file format, and the unusual sender domain were all identified as suspicious.

**Verification attempt.** Before fully trusting the workflow, an attempt was made to obtain a legitimate Microsoft Teams calendar invitation or a `teams.microsoft.com` direct link. The attacker's refusal to provide one served as the final confirmation of malicious intent.

**Evidence preservation.** The full email chain, screenshots of every stage of the fake Teams experience, the Cloudflare phishing warning, the WinSec artifact folder, and the malicious JavaScript source were all preserved before remediation. This is the right order of operations: document first, then clean.

**Appropriate remediation decision.** Choosing a clean Windows reinstall rather than relying only on antivirus was the correct call given the ScreenConnect payload and unknown MSI execution scope. When the extent of a compromise is uncertain, restoring from clean is the right answer.

---

## What Could Have Gone Differently

**Domain validation before opening the link.** The meeting link URL (`ms-teaminvite.team`) did not contain `microsoft.com` or `teams.microsoft.com`. A quick check of the URL before clicking would have revealed the mismatch immediately. Any legitimate Microsoft Teams meeting URL begins with `teams.microsoft.com`.

**Publisher signature check before execution.** The SmartScreen "unknown publisher" warning is a meaningful signal. Legitimate Microsoft installers are signed by Microsoft Corporation. An unsigned file claiming to be a Microsoft installer is a strong indicator of malice. Pausing to right-click the file and check its digital signature properties would have confirmed no legitimate signature was present.

**File extension awareness.** Legitimate software installers arrive as `.exe` or `.msi` files. A `.js` file is never a legitimate Teams installer. This was a clear indicator that could have stopped execution before it started.

**Recruiter domain verification.** Before responding to the initial email, verifying that `officesdesknote.com` had any connection to Mercer Bucks Technology would have revealed the mismatch. A quick search for Mercer Bucks Technology's actual domain and contact information takes under two minutes.

**Job posting cross-reference.** The Indeed listing disappeared after initial contact. Checking whether the listing was still active and whether Mercer Bucks Technology's official website or LinkedIn had any matching open positions would have been a useful verification step.

---

## Key Takeaways for Job Seekers

1. **Verify the recruiter's domain against the company's official website.** If a recruiter emails from a domain that does not match the company's official domain, treat it as suspicious.

2. **All legitimate Microsoft Teams meeting links begin with `teams.microsoft.com`.** Any meeting link using a different domain is not from Microsoft.

3. **Microsoft Teams does not deliver updates as `.js` files from non-Microsoft websites.** If a website claims Teams needs an update and offers a JavaScript file, it is malware.

4. **SmartScreen warnings on installer files matter.** If Windows says the publisher is unknown, check the file's digital signature before proceeding. Legitimate Microsoft software is signed.

5. **Excitement and urgency are social engineering tools.** Recruitment phishing works because job seekers want the opportunity to be real. Threat actors know this. Slow down and verify.

6. **If you cannot obtain a legitimate calendar invitation or official meeting link, do not proceed.** A real recruiter from a real company can always provide a meeting link from their company's own domain or from `teams.microsoft.com`.

---

## Key Takeaways for Security Practitioners

1. **Recruitment phishing is a growing and underreported vector.** Job seekers, especially those in technical fields, represent a high-value target population because they often have technical access and are conditioned to accept tools and installation requests during onboarding workflows.

2. **ScreenConnect is increasingly abused as a payload.** As a commercial, signed remote access tool, ScreenConnect can bypass some security controls and is less likely to trigger antivirus alerts than custom RAT implants.

3. **Telegram-based C2 is highly evasion-resistant.** Traffic to `api.telegram.org` looks identical to legitimate Telegram usage and will not be flagged by domain reputation tools that have not specifically categorized Telegram API traffic as suspicious.

4. **Cloudflare R2 for payload hosting is an emerging technique.** Hosting malware on legitimate cloud storage infrastructure (R2, S3, Azure Blob) bypasses URL category filters that block unknown domains.

5. **WSH JScript requires no elevated privileges for download and execution.** Any Windows user can execute `.js` files via WSH. No admin rights are required for the dropper stage.

---

## Awareness Recommendations

For individuals:
- Enable Safe Links / URL scanning in your email client if available
- Bookmark the real Teams URL: `teams.microsoft.com`
- Treat any `.js` file in a download as immediately suspicious
- Cross-reference all recruiter emails against the company's verified LinkedIn or official website

For organizations:
- Consider blocking execution of `.js` files via Windows Script Host using Group Policy (`HKCU\Software\Microsoft\Windows Script Host\Settings\Enabled = 0`)
- Monitor for `WinHttp.WinHttpRequest` or `ADODB.Stream` usage in script engines via EDR telemetry
- Alert on outbound connections to `api.telegram.org` from non-browser processes
- Consider monitoring for MSI installations from `%TEMP%` directory
- Alert on new folder creation in `%USERPROFILE%\Documents\` with names matching security-themed keywords (WinSec, WinLog, etc.)
