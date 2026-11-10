# Zeek Network Security Monitoring — investigating torrent activity from a packet capture

**SecOps Detection · 2026-11-05 · Environment: isolated lab**

## What this lab was about

Take a single packet capture where one host was flagged for unauthorized torrent activity, and confirm or rule it out. Use Zeek to turn the raw capture into per-protocol logs, then answer the basic investigation questions one field at a time: which machine, which user, which OS, what was downloaded, when, and with what software. Same "know normal, find evil" logic as the other SecOps labs, but on network logs instead of host logs.

## Tools I used

- Zeek — reads a pcap and splits it into per-protocol logs (conn, dns, http, kerberos, dhcp)
- zeek-cut — pulls columns out of a Zeek log by field name instead of counting positions
- grep / awk / egrep / sort / uniq — filter and clean the output
- tshark — command-line Wireshark, used to decode a binary field Zeek couldn't show cleanly

## What I did

I put the zeek binary on my PATH and ran it against the capture to generate the logs. From there the work was a chain of small questions, each answered by cutting the right field out of the right log. I started in the DHCP log to get the host's MAC and hostname, carried that hostname into the Kerberos log to find the user account, dropped the machine-account lines to isolate the human one, then read the HTTP log for the OS (User-Agent), the torrent hosts, and a readable timeline. The torrent info_hash was stored as escaped binary, so I switched to tshark to read it straight from the pcap.

```bash
# isolate the user account behind the host in the Kerberos log
cat kerberos.log | zeek-cut id.orig_h client service | awk '$3~"krbtgt"' | grep -vi <hostname>
```

## What I found

The host resolved to one machine (hostname, IP, MAC) running Windows 10, tied to a single user account. The HTTP timeline showed torrent traffic, the client was Deluge, and the extracted info_hash matched the official ubuntu-18.04-desktop-amd64 image. One detail mattered: the announce requests showed downloaded=0 and event=started, so the transfer had only just begun. I could prove the download was initiated, not that it finished.

## What I learned

One piece of evidence is a lead, not a verdict. A clean Ubuntu ISO hash looks harmless, but the real issue isn't what was downloaded — P2P traffic is often against policy regardless of content, and the same host also searched generic torrent sites. Whether it's an incident depends on policy and the full picture, not a single field. I also learned the same field tells different stories in context: a User-Agent is a browser during web traffic but names the torrent client during P2P traffic. And I learned where Zeek stops and tshark starts — Zeek triages who/what/when fast across a lot of traffic, but a raw binary field needs packet-level decoding. "The data doesn't show that" is a valid answer: it proved the download started, not that it completed.

## How I'd defend against it

Start at the policy and network layer, not the files. If P2P isn't allowed, block BitTorrent at the firewall or proxy and alert on tracker traffic — announce and scrape requests are a reliable signal, and so is a non-browser User-Agent naming a torrent client. This is exactly the kind of behavior worth encoding as an IDS rule so the sensor catches it live instead of someone finding it in a pcap later. Tying the host to a named user through Kerberos is what turns "an IP did something" into "this account did something," so authentication logging is part of the defense, not just the investigation. And none of this works without packet capture and log retention covering the window an analyst actually needs.

## Ties to Security+

Security Operations (log analysis, timeline building, correlating events across sources), network security monitoring and IDS, indicators and threat context, and alerting on behavior (tracker traffic, client User-Agents) rather than a single artifact. The networking underneath — DHCP, DNS, Kerberos, HTTP User-Agents, and how BitTorrent announces to trackers — is the supporting knowledge that makes the fields make sense.
