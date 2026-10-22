# Binary Static Analysis — brbbot.exe

**Module:** Threat Hunting & Incident Response
**Tools:** BinText, PEStudio, Detect It Easy (DIE), VirusTotal (by hash)
**Sample:** brbbot.exe (real malware, PE 64-bit)

## What this lab was about

Looking at a real piece of malware and figuring out if it's dangerous — without ever running it. That's what static analysis is: you read the file while it sits still. It's the first of the three phases of malware analysis (static, then dynamic, then reverse engineering).

The one rule I kept the whole time: read it, never run it.

## Tools I used

- **BinText** — pulls the readable text (strings) out of the file
- **PEStudio** — the main triage tool: hash, indicators, libraries, imports, MITRE ATT&CK tags
- **Detect It Easy** — compiler info and the entropy graph (tells you if the file is packed)
- **VirusTotal** — reputation check, searched **by hash** so I never upload the sample

I did all of this on an isolated Windows VM, with the antivirus turned off for a moment (otherwise Defender deletes the sample), and turned back on when I was done.

## What I did

1. Got the file's hash and basic PE info — the hash is the file's identity.
2. Pulled the strings with BinText to get a first idea of what it touches.
3. Loaded it in PEStudio to see the indicators, the DLLs, and the imported functions.
4. Checked if it was signed.
5. Checked the entropy in DIE to see if it was packed or encrypted.
6. Compared it to a legit program to understand the limits of static analysis.

## What I found

The file is a PE executable, 64-bit, **not signed**, entropy **5.9 (not packed)**.

SHA-256: `F47060D0F7DE5EE651878EB18DD2D24B5003BDB03EF4F49879F448F05034A21E`

The strings already told a story:

| String | What it means |
|---|---|
| `Mozilla/4.0 (...MSIE 8.0...Trident/4.0)` | fake user-agent to blend into web traffic |
| `%s?i=%s&c=%s&p=%s` | URL pattern for talking to a C2 server |
| `...CurrentVersion\Run` | registry key for startup persistence |
| `brbconfig.tmp` | config file it drops |

The imports confirmed it at the code level: network functions (`HttpSendRequestA`, `InternetConnectA`), registry functions (`RegSetValueExA`), and resource functions (`LoadResource`). PEStudio flagged 43 of the imported functions and two network DLLs (`wininet.dll`, `ws2_32.dll`).

So three different sources — the strings, PEStudio's tags, and the import table — all point to the same behavior: a bot that phones home and sets itself to start with Windows.

## What I learned

- **One clue doesn't prove anything.** You stack the signals (strings + imports + signature + entropy + VirusTotal) and judge the whole picture, with the context in mind.
- **Static tells you what a file *can* do, not what it *does*.** For the "what it actually does" part, you need dynamic analysis.
- **High entropy doesn't mean bad.** A legit installer like Firefox is packed (entropy ~7.9) and shows more "severe" indicators than this malware. What really separates them is the signature and the VirusTotal reputation, not the raw indicator count.
- The hash is the file's identity for threat intel, but it's easy to change by recompiling — so I never trust it alone.
- Small connection to another lab: the `MZ` header (4D 5A) of a PE file is the same marker `malfind` was hunting for in memory during the Stuxnet lab.

## How you'd defend against it

- Detection rules on the user-agent and the C2 URL pattern.
- Hunt the `Run` key and the `brbconfig.tmp` file across the fleet.
- Search for the hash in the EDR/SIEM.
- If static analysis isn't enough to decide, move it to a sandbox for dynamic analysis.

## Ties to Security+

Malware analysis and IoCs, static vs dynamic analysis, packing/obfuscation, code signing and hashing, threat intelligence, MITRE ATT&CK.
