# Email Header Analysis — reading an email's routing path and checking SPF/DKIM to decide if it's legit

**Social Engineering · 2026-10-31 · Environment: isolated lab**

## What this lab was about

An email header is the hidden record of every mail server a message passed through on its way to me. This lab was about reading that record: following the path the message took, and checking the authentication results (SPF, DKIM, DMARC) to judge whether the message is trustworthy. The main idea I took away is simple — don't trust the "From" line, trust what you can actually verify.

## Tools I used

- A plain text editor — to read the raw header of a sample message
- nslookup — to check a domain's DNS TXT records myself instead of trusting the header
- The authentication fields the receiving mail server had already stamped into the header

## What I did

I opened a sample email's full header and worked out its structure — a formatted block on top, the raw header below — and read it the way headers are meant to be read: bottom to top, since each server adds its block above the previous ones. Starting at the `To:` line, I followed the path upward through each `Received:` line, breaking one of them into its parts (`from` the sender of that hop, `by` the receiver, `with` the technology, `via` the service). I read the `Authentication-Results:` line for the SPF/DKIM/DMARC verdicts, but didn't take `spf=pass` on trust: I looked up the sender domain's SPF record myself, saw it only pointed to another record with an `include:`, followed that, and confirmed the sending IP fell inside an approved range. Finally I cross-checked the `From`, `Return-Path`, and `Message-ID` domains against each other, and read the spam stamps (SCL, BCL) the receiving server had left.

Key command (neutralized):

```bash
nslookup -query=txt example-sender.com
# -> "v=spf1 include:_spf.example-sender.com ~all"   (no IPs here — follow the include)
nslookup -query=txt _spf.example-sender.com
# -> "v=spf1 ip4:<approved-range>/24 ... ~all"        (the sending IP falls inside this range)
```

## What I found

The `Received:` lines chained correctly — the `by` server of one hop became the `from` server of the next, a clean chain of custody. SPF, DKIM, and DMARC all passed, and I confirmed SPF myself against DNS rather than trusting the header. `From`, `Return-Path`, and `Message-ID` all used the same domain, with no mismatch. One thing stood out: the real DNS records didn't match the course material — the provider had added more IP ranges since the lab was written, because the DNS is live and the doc was only a snapshot. Verdict: a legitimate transactional message (a verification code), not phishing.

## What I learned

Headers read bottom-up — the first server in the journey is the bottom `Received:` line, not the top one, and that was the least intuitive part, the thing I had to slow down on. SPF, DKIM, and DMARC answer three different questions: SPF asks whether this IP is allowed to send for the domain (checked against a DNS record), DKIM whether the signature proving origin and integrity is valid (verified with a public key in DNS), and DMARC whether the authenticated domain lines up with the visible `From` — and what to do if it doesn't. A passing SPF is not proof a message is safe: in the spam examples I looked at, SPF still passed, but the `From` / `Return-Path` / `Message-ID` domains disagreed and the transit timestamps were impossible. No single field decides it; the signals have to agree. The receiving edge server (the public MX) is what runs these checks, because it's the only point that sees the real sending IP — once the message is inside the organization, the internal hops trust each other. And I learned to verify against live DNS, not a value I memorized or read in a doc: all three of my lookups came back different from the course material. (Smaller takeaway: encoded header words like `=?us-ascii?Q?...?=` come from RFC 1342 — the middle letter is the encoding, Q for quoted-printable or B for base64, and the charset is separate, so "us-ascii" there is the charset, not the encoding.)

## How I'd defend against it

Publish correct SPF, DKIM, and DMARC records for my domains, and move DMARC to a real enforcing policy (quarantine or reject) once alignment is confirmed, rather than leaving it at `p=none` forever. Alert on authentication failures, and just as much on domain mismatches between `From`, `Return-Path`, and `Message-ID`, since those catch spoofing that still passes SPF. Treat the `From` line as untrusted display text, and base trust on the authenticated, aligned domain, not on what the message claims. Flag impossible transit times and broken `Received:` chains as spoofing indicators. And train users that an urgent "verification code" or "package tracking" email from a mismatched sender is the classic lure — with a one-click report button so the signal reaches the security team.
