---
title: "Lab 5: UNION Attack — Retrieving Data from Other Tables"
---
# Lab 5: SQL Injection UNION Attack — Retrieving Data from Other Tables

**Platform:** PortSwigger Web Security Academy

## What's going on here

Everything so far — column count, finding a text column — was setup. This is where it pays off: using UNION to pull real data out of a table the app was never designed to expose, straight into the response you already control.

## 🎯 Target

Retrieve usernames and passwords from the `users` table.

## The bypass

First, confirmed the column layout: two columns, both text-capable.

'+UNION+SELECT+'abc','def'--


![Confirming two text-capable columns](images/lab5-step1-column-check.png)

With that confirmed, swapped the placeholders for a real query against the `users` table — pulling `username` into the first column and `password` into the second:

'+UNION+SELECT+username,+password+FROM+users--


![Payload retrieving usernames and passwords](images/lab5-step2-payload.png)

Since both columns already accept text, there was no need to concatenate values into one slot — each one just landed in its own column directly.

## Result

The response came back listing usernames and passwords straight from the `users` table.

![Usernames and passwords visible in the response](images/lab5-step3-success.png)

## Takeaway

This is the actual payoff of a UNION attack — not just proving the injection exists, but using it to walk out with real credentials from a table that was never meant to be reachable through this query. Column count and text-column discovery aren't busywork; they're the reconnaissance that makes this step possible.

---

Credentials: extracted, two clean columns at a time. But not every query is this generous — some only expose a single usable text column, meaning multiple values have to get squeezed into one slot: **[Lab 6 — SQL Injection UNION Attack, Retrieving Multiple Values in a Single Column](lab-6-multiple-values-column.md)**.
