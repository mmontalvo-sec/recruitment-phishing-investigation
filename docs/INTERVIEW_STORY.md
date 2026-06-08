# Interview Story

**Question:** "Tell me about a time you dealt with a security incident" or "Tell me about a real-world experience that involved threat identification, containment, and documentation."

---

## Short Version (60 seconds)

"Earlier this month I encountered a multi-stage phishing campaign disguised as a job interview. The threat actor sent me a convincing recruiter email, followed by what looked like a Microsoft Teams meeting invite. When I opened the link, it redirected to a non-Microsoft domain running a cloned Teams interface, complete with camera and microphone prompts.

The site then presented a fake update prompt and delivered a JavaScript file. Through static analysis of the recovered script, I identified that it was a Windows Script Host dropper that downloaded a ScreenConnect remote access client disguised as a Microsoft Teams installer, and used a Telegram bot to exfiltrate my hostname, username, and public IP in real time. The endpoint also produced PIN-style keystroke logging artifacts.

I isolated the machine, preserved all available evidence, removed what I could manually, ran a full Defender scan, and completed a clean Windows reinstall to restore full endpoint trust. I documented everything: email chain, screenshots, IOCs, a full incident report, MITRE ATT&CK mapping, and a lessons learned document. That investigation is now a public portfolio case study."

---

## Long Version (90 seconds)

"I was in the middle of a job search when I received what appeared to be a legitimate interview invitation from a company I had actually applied to through Indeed. The recruiter email, the scheduling back-and-forth, and the Teams meeting invite all looked realistic. But when I clicked the join link, the URL in the address bar was `ms-teaminvite.team`, not anything from Microsoft.

The page was a nearly perfect clone of the Microsoft Teams web pre-join experience. It requested camera and microphone permissions, showed a Teams-branded interface, and then told me Teams needed an update. It downloaded a JavaScript file, which is not how Microsoft delivers updates.

I ran it, which I recognized as a mistake as soon as I saw anomalous behavior on the machine. I immediately started collecting evidence: screenshots of every stage, the email chain, the page source, and most importantly, I preserved the JavaScript file itself. Static analysis showed it was a dropper that pulled down a ScreenConnect MSI from Cloudflare R2, disguised as a Teams installer, while simultaneously sending my hostname, username, and public IP to an attacker-controlled Telegram bot. I also found local log files in a folder called WinSec that appeared to capture PIN keystrokes.

I isolated the endpoint, attempted manual removal, identified a ScreenConnect DLL remnant in Program Files, ran a full Defender scan, and performed a clean Windows reinstall. Then I documented the full incident including a timeline, IOC list, MITRE ATT&CK mapping with confidence levels, and a detection opportunities document outlining where controls could have flagged it earlier.

The whole thing is published as a sanitized public case study on my GitHub. The outcome was a clean, trusted endpoint and a documented real-world incident investigation."

---

## Key Points to Hit (for flexible delivery)

- **What it was:** Recruitment-themed spearphishing campaign, not a generic email
- **What I noticed:** Domain mismatch, JavaScript file format, attacker's refusal to provide a legitimate link
- **What I found technically:** ScreenConnect payload, Telegram C2, PIN keystroke logging artifacts
- **What I did:** Isolated, preserved evidence, static analysis, containment, clean reinstall
- **What I produced:** Full incident report, timeline, IOCs, MITRE mapping, lessons learned, public case study
- **Tone:** Calm, technical, analytical. Not "I got attacked." Frame it as "I investigated and documented a sophisticated campaign."

---

## Handling the Obvious Follow-Up: "Did you run it?"

Be direct and honest. Do not be defensive.

"Yes, I executed it. The social engineering was convincing: a real job application context, a realistic recruiter workflow, and a professionally cloned Teams interface. The mistake was bypassing the SmartScreen publisher warning. That is now permanently part of my personal SOP: any installer that does not carry a verified publisher signature gets stopped, full stop.

What I did well was recognizing the anomaly quickly, pivoting immediately to evidence collection and containment, and treating it as an investigation rather than a crisis. I learned more from that incident than I would have from any lab exercise."

---

## Recruiter-Safe One-Liner

"I investigated a sophisticated recruitment phishing campaign that used a cloned Microsoft Teams interface and a JavaScript dropper to deliver ScreenConnect remote access software, contained the endpoint, performed static analysis of the payload, and documented the full incident including IOCs and MITRE ATT&CK mapping."
