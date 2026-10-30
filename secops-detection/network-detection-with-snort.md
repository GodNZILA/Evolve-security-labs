# Network Detection with Snort — writing custom detection rules and tuning out the noise

**SecOps Detection · 2026-10-29 · Environment: isolated lab**

## What this lab was about

Deploy Snort as a network IDS on an Ubuntu host, configure it for its own perimeter, then write and tune custom rules that detect specific traffic in real time: ping, SSH in and out, HTTP GET/POST, port scans, and a hostile-IP list. The point is to take "know normal, find evil" and encode it into rules so the machine catches the anomaly live, then cut the false positives.

## Tools I used

- Snort (NIDS) on Ubuntu
- nano — edit the config and rules
- curl / nc / nmap — generate the test traffic from a second machine

## What I did

I installed the dependencies (one library for packet capture, two for parsing rules), set HOME_NET in both config files so it stays consistent whether Snort runs as a service or by hand, and validated the config before running anything. Then I wrote rules in the local rules file (custom rules use IDs above one million), ran Snort in console alert mode, and tested each rule by generating the matching traffic from the attacker box.

One rule, inspecting the payload instead of just the envelope:

```
alert tcp any any -> $HOME_NET <port> (msg:"POST seen"; content:"POST"; sid:1000004;)
```

## What I found

The whole chain fired live — ICMP, inbound and outbound SSH, GET and POST (payload inspection), a port scan, and a rule matching a list of hostile IPs. The real finding was tuning: my first SSH rule was too broad (any source) and kept matching my own admin session — pure white noise. I tightened it to the attacker's source address and the signal came back.

## What I learned

Where $HOME_NET sits around the arrow decides direction — unexpected outbound SSH is a red flag for a pivot or exfiltration. Tuning is the actual job: a rule that alerts constantly stops being read, so narrowing it is what turns noise into signal. You detect the attempt, not just the result — the alert fires even when the action fails, and failed attempts are often the first sign of an attack. Canary ports with no legitimate service give almost no false positives, so any hit there means something.

## How I'd defend against it

This lab is the defense, but the lesson is that detection only works if it's operated well. Pair signature rules (what's in the payload) with reputation lists (who's talking), watch for direction anomalies like outbound admin protocols, and start in alert mode before any inline blocking. An un-tuned sensor is worse than none, because nobody reads an alert that's always firing.

## Ties to Security+

Security Operations (IDS/IPS, detection and alert tuning, false positives, baseline vs anomaly), threat intelligence (indicators and reputation lists), signature-based vs reputation-based detection, and ports/protocols (ICMP, SSH, HTTP).
