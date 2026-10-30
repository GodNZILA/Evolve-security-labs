# Phishing Campaign with GoPhish — Part 1: building the infrastructure

**Social Engineering · 2026-10-23 · Environment: controlled awareness exercise**

> Ethics: this was an awareness exercise on a throwaway domain I controlled, with no real targets. A real engagement requires written authorization and a defined scope.

## What this lab was about

Build, end to end, the infrastructure for a phishing campaign with GoPhish, to understand what actually makes an email credible: the domain, and the DNS records (SPF/DKIM) that get it past spam filters. Part 1 is the setup — a cloud server, GoPhish, a domain, a mail relay, and DNS. Sending the campaign itself is Part 2.

## Tools I used

- GoPhish — the phishing framework
- A cloud VM (Ubuntu) — to host it
- A domain registrar and a free mail provider — domain and mail relay
- ssh / scp / nslookup — deploy and verify

## What I did

I launched the cloud instance with a security group opening only the ports I needed, copied the installer over scp, installed GoPhish, and changed its admin listener from localhost to all interfaces so I could reach the console. Then I registered a disposable domain and set up the DNS: an A record pointing the domain at the server, MX records for mail, an SPF record authorizing the mail provider, and a DKIM record (the RSA signature). I verified every record from the command line before trusting the web interface.

The check I repeated for each record:

```
nslookup -type=txt <my-domain> 8.8.8.8
```

## What I found

The chain domain → DNS → server worked: hitting the domain returned GoPhish's default 404, which is the success signal — the server is reachable, there's just no landing page yet. SPF and DKIM both resolved and verified.

## What I learned

DNS is the root of trust — controlling it is what proves you own the domain, and it's the shared basis of the ownership check, SPF and DKIM. SPF says who may send, DKIM is the signature (origin and integrity), DMARC says what to do on failure — all bolted onto SMTP, which verifies nothing by itself. A port open in the security group isn't enough if the service only listens on localhost; both have to be true. And the irony of the lab: the same anti-spoofing records that protect real companies are exactly what make a phishing email credible when they're set up properly.

## How I'd defend against it

Don't just publish SPF and DKIM — enforce DMARC with a quarantine or reject policy so spoofed mail is actually dropped. Monitor for look-alike domains, and teach users that a technically "authenticated" email can still be hostile: authentication proves the sender controls a domain, not that they can be trusted.

## Ties to Security+

Social engineering (phishing, spear-phishing, whaling), email security (SPF, DKIM, DMARC), DNS and record types, secure protocols (ssh/scp), cloud IaaS, and certificates (the self-signed TLS warning).
