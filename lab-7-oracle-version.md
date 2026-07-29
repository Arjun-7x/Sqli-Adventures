# Lab 7: SQL Injection Attack — Querying the Database Type and Version on Oracle

**Platform:** PortSwigger Web Security Academy

## What's going on here

So far, every attack has targeted the application's own data — products, users, credentials. This one's different: instead of pulling data the app owns, the target is the database engine itself. Knowing exactly which database and version is running (Oracle, in this case) tells an attacker what syntax, functions, and quirks are available for everything that comes next.

## 🎯 Target

Retrieve the database version string from an Oracle backend.

## The bypass

First, confirmed the column layout: two columns, both capable of holding text.

'+UNION+SELECT+'abc','def'+FROM+dual--


![Confirming two text-capable columns via Oracle's dual table](images/lab7-step1-column-check.png)

`dual` is worth noting — Oracle requires every `SELECT` to reference a table, even when there's nothing real to select from. `dual` is a built-in dummy table that exists purely so a query like this has something to point at.

With the column layout confirmed, swapped in the real target — Oracle's `v$version` table, which stores the database's version banner:

'+UNION+SELECT+BANNER,+NULL+FROM+v$version--


![Payload returning the Oracle version banner](images/lab7-step2-payload.png)

## Result

The response came back with the full Oracle version string sitting right in the page.

![Oracle version banner visible in the response](images/lab7-step3-success.png)

## Takeaway

Fingerprinting the database isn't the flashy part of an attack, but it's often the first real move once injection is confirmed — it shapes every payload that follows. Oracle's `v$version`/`dual` quirks won't carry over to MySQL or Microsoft SQL Server, which is exactly what the next lab tests.

---

Oracle's given up its version. Next: same goal, different engines — MySQL and Microsoft SQL Server each have their own way of answering this question: **[Lab 8 — SQL Injection Attack, Querying the Database Type and Version on MySQL and Microsoft](lab-8-mysql-mssql-version.md)**.
