# Phishing Campaign with GoPhish — Part 2: launching and measuring the campaign

**Social Engineering · 2026-10-23 · Environment: controlled awareness exercise**

> Ethics: awareness exercise on a throwaway domain, with targets limited to my own addresses and consenting volunteers. Any password capture was against a disposable account only.

## What this lab was about

On the Part 1 infrastructure, assemble and launch a full phishing campaign in GoPhish and measure it — opens, clicks, submissions. The goal was to see how the credible email, the fake landing page and the tracking fit together, and how a campaign is actually run and read.

## Tools I used

- GoPhish — sending profile, template, landing page, campaign
- The mail provider's SMTP (over SSL) — to send
- curl — to diagnose connectivity

## What I did

I built the five pieces of a campaign: a sending profile (SMTP over SSL, using the mail account's credentials, not GoPhish's), a fake landing page whose form captures the submitted data and then redirects to an awareness page, an email template using GoPhish's tags for personalization plus the tracking pixel and the trap link, a target group, and the campaign itself. Then I launched it and watched the dashboard.

When "Send Test Email" hung on an endless spinner, I checked the SMTP path rather than assuming a bad password:

```
curl -v telnet://<smtp-host>:465 --max-time 10
```

## What I found

The campaign ran and showed up live on the dashboard — opens, clicks and submissions tracked per target. The spinner turned out to be a dropped local connection mid-send, not a credentials problem: the curl check reported the connection established, so once I restored the link the test mail went out.

## What I learned

An infinite spinner is a connectivity or timeout problem, not a password one — a wrong password fails fast, so the symptom itself tells you where to look. The display name is not the real address: the address in the brackets has to be a real mailbox on the sending domain. The tracking tag measures opens; the per-target link measures clicks. And ethics is part of the method, not an afterthought — capture credentials only when mandated, and always redirect to an educational page.

## How I'd defend against it

Measure the human layer the way an attacker would: run authorized awareness campaigns, track click and submit rates, and follow up with training rather than blame. On the technical side, rewrite or strip links at the mail gateway, clearly flag external senders, and make reporting a suspicious email a single click for users.

## Ties to Security+

Social engineering, email security (SPF/DKIM/DMARC), security awareness training, and security operations (phishing simulation and its metrics).
