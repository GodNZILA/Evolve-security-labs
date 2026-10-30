# Windows Event Log Analysis — reconstructing a full intrusion from Windows logs alone

**SecOps Detection · 2026-10-29 · Environment: isolated lab**

## What this lab was about

Rebuild a complete intrusion using only the logs Windows records on its own, in Event Viewer, with no third-party tools. The idea is "know normal, find evil": tie isolated events into one timeline, then map each key moment to MITRE ATT&CK. It's the same logic as a memory investigation, but on the traces the OS keeps by itself instead of on RAM.

## Tools I used

- Event Viewer (native Windows) — load and filter the saved logs
- MITRE ATT&CK — map each step to a technique
- A public Event ID reference — to confirm what each ID means

## What I did

I loaded three saved logs: RDP connections, the Security log (the important one — logons, accounts, groups), and the System log for service installs. I filtered by the Event IDs that matter (service install, account created, added to a group, failed and successful logon, explicit-credentials logon, logoff) and followed the account SID when the account name was blank. On each successful logon I read the Logon Type, and when a workstation name looked spoofed I opened the event's XML view to get the real one.

## What I found

The timeline: the attacker installed a malicious service running as LocalSystem to get SYSTEM, then created a local admin account and added it to both Administrators and Remote Desktop Users to persist, probed a disabled account with failed logons, connected over RDP, and replayed stolen credentials with runas. The General tab showed a spoofed workstation name on the RDP logon; the XML of a network-type logon revealed the attacker's real host. Verdict: the malicious service was the root cause, the RDP session only a consequence.

## What I learned

The Logon Type is the key to the logon event — the same event tells a local, network or RDP story depending on that one field. No single event proves anything; the chain does. The raw layer doesn't lie even when the presented view shows a spoofed name, so when a field looks wrong I drop to the XML. And "insufficient data" is a professional answer: the logs show what happened, not always why.

## How I'd defend against it

Centralize logs in a SIEM and alert on the pattern instead of hunting by hand — for example, a new account added to a privileged group within minutes, on the same SID. Restrict who can install services and who can RDP, and alert specifically on service installs running as SYSTEM. The attack is only visible because Windows logged it, so the first control is making sure those logs are collected and kept off the host.

## Ties to Security+

Security Operations (log analysis, SIEM correlation, timeline building), incident response, MITRE ATT&CK (Initial Access, Execution, Persistence, Credential Access, Privilege Escalation), and Windows logon types.
