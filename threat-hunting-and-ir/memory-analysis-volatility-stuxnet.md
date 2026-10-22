# Memory Analysis with Volatility — Stuxnet

**Module:** Threat Hunting & Incident Response
**Tools:** Volatility 3, Kali Linux
**Target:** a RAM memory dump from a Windows machine infected with Stuxnet

## What this lab was about

Finding a malicious process inside a memory dump. The idea is simple: you need to know what *normal* looks like before you can spot what's wrong. Memory is a good place to catch malware because that's where it drops its guard — decrypted, injected, sockets open — things you wouldn't see on disk.

## Tools I used

- `windows.info` — OS context so the parsing is correct
- `windows.pstree` / `windows.pslist` — the process tree, parents, create times
- `windows.dlllist` — how many DLLs each process loads
- `windows.malfind` — finds injected code (executable + writable memory regions)

## What I did

1. Pulled the OS context first.
2. Looked at the process tree and noticed something off: three `lsass.exe` when there should be one.
3. Checked the parent of each one.
4. Compared the three instances — DLL count, create time, sockets.
5. Ran `malfind` to look for injected code in memory.

## What I found

| Instance | Parent | DLLs | Injected PE (RWX) | Verdict |
|---|---|---|---|---|
| lsass #1 | winlogon.exe (normal) | ~61 | no | legit |
| lsass #2 | services.exe (wrong) | ~12 | yes (MZ) | malicious |
| lsass #3 | services.exe (wrong) | ~32 | yes (MZ) | malicious |

`services.exe` should never be the parent of `lsass` — they're peers. The two fake ones load very few DLLs and carry an injected PE in memory regions that are both writable and executable. No single clue proves it, but put together it's clear.

## What I learned

- **Know normal, find evil.** The whole thing only works because I knew what to expect: one `lsass`, parent is `winlogon`, lots of DLLs, no RWX memory.
- **The parent/child link is the first thing I check.** A core process with the wrong parent is an immediate red flag.
- **RWX + an MZ header = injection.** A normal executable is read + execute, never writable. Writable and executable memory holding a PE header is a classic injection sign.
- **DLL count only means something by comparison** with a healthy instance, not as a fixed number.
- Malware often steals the name of a critical system process to hide in plain sight.

## How you'd defend against it

- Baseline the expected parent and DLL count of system processes so anomalies stand out.
- Alert in the EDR on any RWX memory region that contains a PE header.
- Collect memory in the right order of volatility during an incident.
- Hand the IoCs and the process dump to whoever does the reverse engineering.

## Ties to Security+

Incident response and forensics (order of volatility, memory acquisition), process injection and masquerading, IoCs, the kill chain.
