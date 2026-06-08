# LinkedIn Post

---

## Draft (Full Version — Recommended)

---

A few days ago, I investigated a phishing campaign designed to target IT job seekers.

I had applied for an IT Support Specialist role through Indeed. Shortly after, I received a recruiter email, a scheduling exchange, and a Microsoft Teams meeting invitation. The workflow was convincing. The branding looked right. The timing felt normal.

But the meeting link did not go to `teams.microsoft.com`.

It redirected to a threat-actor-controlled domain running a pixel-accurate clone of the Microsoft Teams web pre-join experience — camera and microphone permissions, audio device selection, a name field, a "Join now" button. The whole thing.

Then the page said Teams needed an update and prompted a download.

The file was a `.js` file.

That is not how Microsoft delivers updates.

I ran it. The social engineering was well-constructed and I was in a job search mindset where urgency and opportunity lower your guard. I recognized the anomaly quickly, stopped, and pivoted to evidence collection and containment.

**What I found through static analysis of the recovered script:**

The file was a Windows Script Host JScript dropper that:

- Downloaded a ScreenConnect remote access client disguised as `Microsoft Teams.msi`
- Collected my hostname, username, public IP, and operating system via WMI and environment variables
- Reported execution status in real time to an attacker-controlled Telegram bot
- Used Cloudflare R2 cloud storage as the payload host to bypass domain reputation filters

The endpoint also produced local artifacts consistent with a PIN-style keystroke capture component.

**What I did:**

- Isolated the endpoint
- Preserved all evidence: email chain, screenshots, malicious script, local artifacts
- Performed static code review
- Manually removed what I could; identified a ScreenConnect DLL remnant
- Ran a full Microsoft Defender scan
- Performed a clean Windows reinstall to restore endpoint trust
- Documented everything: incident report, timeline, IOCs, MITRE ATT&CK mapping, detection opportunities, and lessons learned

The domain was later flagged by Cloudflare as Suspected Phishing — meaning other people were targeted too.

**The big picture:**

Attackers are targeting job seekers specifically because the emotional context of a job search — urgency, hope, professional credibility — works in their favor. A fake interview invitation is engineered to feel like an opportunity, not a threat.

Red flags to know:
- Legitimate Microsoft Teams meeting links always use `teams.microsoft.com`
- Real software updates are never delivered as `.js` files from a third-party website
- If a recruiter cannot provide a calendar invite from their company domain, that is a signal worth investigating
- SmartScreen "unknown publisher" warnings on installer files are not noise — they are a checkpoint

This incident became a real-world investigation covering social engineering analysis, email forensics, static script review, IOC extraction, endpoint containment, and full incident documentation.

The full case study is on my GitHub. Link in comments.

Every incident is a chance to learn. This one I will not forget.

#Cybersecurity #Phishing #IncidentResponse #SOC #ThreatIntelligence #JobSearchSecurity #MicrosoftTeams #ITSupport #InfoSec #SecurityAwareness

---

## Shorter Version (if you want something punchier)

---

I recently investigated a phishing campaign that targeted me as a job seeker.

The setup: a convincing recruiter email, a scheduling exchange, and a fake Microsoft Teams meeting invite. The meeting link redirected to a cloned Teams interface on a non-Microsoft domain. When the page prompted a "required update," it delivered a malicious JavaScript file.

Static analysis of the recovered script revealed:
- A Windows Script Host dropper
- ScreenConnect remote access software as the payload, disguised as a Teams installer
- Real-time victim telemetry sent to an attacker-controlled Telegram bot
- Cloudflare R2 used for payload hosting to evade reputation filters

I isolated the endpoint, preserved evidence, reviewed the artifacts, and documented the full incident: timeline, IOCs, MITRE ATT&CK mapping, and lessons learned.

The domain was confirmed as suspected phishing by Cloudflare. Other job seekers are being targeted.

If you are actively job searching: verify recruiter domains, never open `.js` files claiming to be software updates, and remember that a real recruiter can always provide a `teams.microsoft.com` link.

Full case study on GitHub — link in comments.

#Cybersecurity #Phishing #IncidentResponse #SOC #JobSearchSecurity #MicrosoftTeams #InfoSec
