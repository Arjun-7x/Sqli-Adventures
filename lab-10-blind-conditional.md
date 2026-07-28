# Lab 10: Blind SQL Injection with Conditional Responses

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

![Boolean true vs false responses side by side](images/lab10-step1-boolean-test.png)

That confirmed the app was silently evaluating my injected condition and changing behavior based on it — even with zero data shown.

**Confirming the target exists:**

TrackingId=xyz' AND (SELECT 'a' FROM users LIMIT 1)='a

True — a `users` table exists.

TrackingId=xyz' AND (SELECT 'a' FROM users WHERE username='administrator')='a

True — an `administrator` user exists.

**Finding the password length** — sent increasing length checks manually in Repeater:

TrackingId=xyz' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>N)='a


Increased `N` until "Welcome back" stopped appearing — the password turned out to be 20 characters.

**Extracting every character in one shot with Cluster Bomb:**

Instead of running 20 separate Intruder attacks (one per position), I used Intruder's **Cluster Bomb** attack type with two payload positions in a single request:

TrackingId=xyz' AND (SELECT SUBSTRING(password,§1§,1) FROM users WHERE username='administrator')='§a§


![Intruder set up with two payload positions for Cluster Bomb](images/lab10-step2-intruder-setup.png)

- **Position 1** — the character offset, a numbered list from `1` to `20`
- **Position 2** — the character guess, a simple list of lowercase letters and digits

Cluster Bomb runs *every combination* of both positions — every offset against every possible character — instead of moving through them one at a time. Set the **Grep - Match** rule to flag `Welcome back` in the response, launch the attack, then filter results for the requests where that column ticked true.

![Intruder results grid, filtered for true matches across offsets and characters](images/lab10-step3-results.png)

Reading down the ticked rows in offset order reconstructed the full 20-character password.

## Result

Extracted the administrator's password character-by-character with zero direct data exposure, then logged in successfully.

## Takeaway

Blind SQLi means the database never hands you anything directly — every bit of information comes from watching a binary signal flip. Cluster Bomb turns what could be 20 sequential attacks (one per position) into a single combinatorial sweep, trading a slightly larger request count for far less manual babysitting.

---

Conditional responses gave away true/false through a visible message. Next: what happens when even that message disappears, and the only signal left is whether the server throws an error at all: **[Lab 11 — Blind SQL Injection with Conditional Errors](lab-11-conditional-errors.md)**.
