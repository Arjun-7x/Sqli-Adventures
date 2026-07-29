---
title: "Lab 11: Blind SQL Injection with Conditional Responses"
---
# Lab 11: Blind SQL Injection with Conditional Responses

**Platform:** PortSwigger Web Security Academy

## What's going on here

Every lab so far had the database talk back — errors, extra rows, version strings, all visible right there in the response. This one doesn't give you any of that. The only signal is a "Welcome back" message that either shows up or doesn't, based on the `TrackingId` cookie. No data leaks directly — but a true/false signal is enough to extract information one bit of certainty at a time.

## 🎯 Target

Determine the administrator's password purely through boolean true/false responses, then log in.

## The bypass

**Confirming the injection is blind but controllable:**

TrackingId=xyz' AND '1'='1

"Welcome back" appeared — condition true.

TrackingId=xyz' AND '1'='2

"Welcome back" disappeared — condition false.

![Boolean true response side](images/lab11-step1-boolean-true.png)
![Boolean false response side](images/lab11-step1-boolean-false.png)

That confirmed the app was silently evaluating my injected condition and changing behavior based on it — even with zero data shown.

**Confirming the target exists:**

TrackingId=xyz' AND (SELECT 'a' FROM users LIMIT 1)='a

True — a `users` table exists.

TrackingId=xyz' AND (SELECT 'a' FROM users WHERE username='administrator')='a

True — an `administrator` user exists.

![Confirming the user administrator account](images/lab11-step2-target-confirmed.png)

**Finding the password length — using Intruder instead of manual Repeater changes:**

TrackingId=xyz' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>§N§)='a


Sent this to Intruder with a single payload position on `N`, loaded with a numeric list from `1` to `22`, and grepped for `Welcome back`. Instead of manually editing and resending 20+ times, one Intruder run showed exactly where the condition flipped from true to false — landing at 20 characters.

![length-Payload-setup](images/lab11-step3-length-check.png)

**Extracting every character in one shot with Cluster Bomb:**

TrackingId=xyz' AND (SELECT SUBSTRING(password,§1§,1) FROM users WHERE username='administrator')='§a§'


![Intruder set up for 1st positions for Cluster Bomb](images/lab11-step4-intruder1-setup.png)

![Intruder set up for 2nd positions for Cluster Bomb](images/lab11-step4-intruder2-setup.png)

- **Position 1** — the character offset, a numbered list from `1` to `20`
- **Position 2** — a brute-force payload set of lowercase letters and digits

Set the **Grep - Match** rule to flag `Welcome back` in the response, same as the length check. Cluster Bomb runs *every combination* of both positions — every offset against every possible character — instead of moving through them one at a time.

![Intruder setting for grep-match](images/lab11-step5-grep-match.png)

Reading down the ticked rows in offset order reconstructed the full 20-character password.

## Result

Extracted the administrator's password character-by-character with zero direct data exposure, then logged in successfully.

## Takeaway

Blind SQLi means the database never hands you anything directly — every bit of information comes from watching a binary signal flip. Running both the length check and the character extraction through Intruder (rather than hand-editing Repeater requests) turns a slow manual process into a couple of automated sweeps.

---

Conditional responses gave away true/false through a visible message. Next: what happens when even that message disappears, and the only signal left is whether the server throws an error at all: **[Lab 12 — Blind SQL Injection with Conditional Errors](lab-12-conditional-errors.md)**.
