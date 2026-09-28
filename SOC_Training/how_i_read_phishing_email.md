How I Read A Phishing Email: Notes From SCO Training
fnwn
fnwn
3 min read
·
Sep 18, 2026





Press enter or click to view image in full size

Hi everyone, fnwn here. If you don’t know who I am, then you can learn about me from fnwn@hacking:~ $whoami.

Write on Medium
Phishing complainings are the most common case I encounter in/from SOC training. A user forward an email, stating “This email looks suspicious” and it is the analyst’s job to evaluate as soon as possible. In this write up, I share the perspective I acquired during my training, even though I have not yet worked on a live case.

Look At The Header First, Not The Body
Directly focusing on email’s content(message, link) is the most common mistake every beginners do. However, to determine whether an email actually originated from the source it claims to be from, the **header** (email header information) is examined first. The header records every servers email pass through and authentication results.

Three Things I Check First
1. Does the “From” Address Match What’s Displayed?
The display name in an email client (e.g., “IT Support”) and the actual sender address (e.g., support@it-helpdesk-login.ru) often don’t match. Trusting the display name without checking the real From field in the header is one of the oldest tricks in phishing.

2. What Do SPF, DKIM, and DMARC Say?
These three concepts confused me most when I first started, but the logic behind them is actually simple:

SPF (Sender Policy Framework) -> Was this email sent from a server the domain owner actually authorized?
DKIM (DomainKeys Identified Mail) -> Was the email altered in transit? (A digital signature check.)
DMARC -> If SPF or DKIM fails, what should happen reject it, send it to spam or let it through?
These three results usually show up in the Authentication-Results line of the header. If all three don’t come back as “pass,” that alone doesn’t confirm malicious intent but it should raise your suspicion level considerably.

3. Where Do the “Received” Lines Say This Came From?
The Received lines in a header list every server the email passed through, from sender to recipient, bottom to top. The bottom-most line shows where the email actually originated. What I pay attention to here: whether that origin point is geographically or organizationally consistent with the domain the email claims to represent.

Reading the URL Before Clicking It
After checking the header, if there’s a link, I hover over it before clicking to read the actual destination. A small but effective trick I picked up here: attackers often use addresses that closely resemble the real domain but with a subtle character swap (like micros0ft-login.com instead of microsoft-login.com). If you know to look for it, it becomes surprisingly easy to spot.

A Simple Decision Schema
Here’s the simplified decision flow I’ve put together from training:

Does the sender address match the display name? -> Mismatch raises suspicion.
Do SPF/DKIM/DMARC all come back “pass”? -> If not, suspicion increases.
Does the link go to the real domain, or a lookalike one? -> A lookalike domain is a strong phishing signal.
Does the language create urgency or threat? (e.g., “Your account will be suspended in 24 hours”) -> A classic social engineering signal.
If more than two of these four come back suspicious, I isolate the email and escalate it.

Where I Stand Right Now
This post summarizes what I’ve learned working through training scenarios, not an actual incident. I haven’t yet had the chance to triage phishing reports in a live SOC environment. As I gain that experience, I plan to share more detailed posts with real examples.

Conclusion
The real skill in phishing analysis is turning “this email feels off” into a decision backed by concrete evidence from the header. That’s the biggest thing SOC training has given me so far: turning instinct into checkable signals.
