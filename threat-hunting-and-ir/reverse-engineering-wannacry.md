# Reverse Engineering — WannaCry

**Module:** Threat Hunting & Incident Response
**Tools:** OllyDbg, strings, network capture (PCAP)
**Sample:** WannaCry (ransomware, 2017)

> Note: I rebuilt this write-up from my notes. I'll flesh it out when I go back through the lab.

## What this lab was about

Taking apart WannaCry to find its famous kill-switch — the hardcoded domain that, when reachable, stops the ransomware from running. This is reverse engineering, the third and last phase of malware analysis, where you read the actual code to understand the logic.

## Tools I used

- **OllyDbg** — the debugger, to read the assembly and follow the branching logic
- **strings** — to spot the kill-switch domain in clear text
- **Network capture (PCAP)** — to see the request going out to that domain

## What I did

1. Found the kill-switch domain in the strings.
2. Used the debugger to find the routine that checks if the domain is reachable.
3. Worked out the branch that decides whether the payload runs or not.
4. Confirmed it on the network side with the outgoing request in the capture.

## What I found

The malware tries to reach a hardcoded domain. If it answers, the encryption payload **does not run**. The branching logic is reversed — registering the domain is what disarms the sample. Three different angles (strings, debugger, PCAP) all pointed at the same domain, which made me confident in the finding.

## What I learned

- **Reverse engineering shows the exact logic** that static and dynamic can only hint at — here, *why* the malware stops.
- **Looking from more than one angle** (code + strings + network) gives a confidence no single angle does.
- One small branch in the code was the weak point that helped stop a worldwide outbreak. Details matter.

## How you'd defend against it

- Patch MS17-010 (the SMB vulnerability it used).
- Disable SMBv1.
- Segment the network around port 445.
- Detect on the IoCs (domain, hash, SMB behavior).

## Ties to Security+

Malware analysis, ransomware, the kill chain, patch management, network segmentation, SMB/445.
