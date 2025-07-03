---
layout: post
title: "Proving Grounds SPX: Privilege Escalation"
---

As I continue to progress through [TJ Null's OSCP PWK V3 list](https://docs.google.com/spreadsheets/u/1/d/1dwSMIAPIam0PuRBkCiDI88pU3yzrqqHkDtBngUHNCw8/htmlview), I wanted to cover a particular machine that had a very unique privilege escalation vulnerability. This machine is given the name SPX. I'll skip over initial access for the sake of brevity and limiting spoilers.

Speaking of spoilers, if you plan on completing this machine and don't want answers given away, DO NOT PROCEED!

# Identifying the Vulnerability

After gaining your initial foothold, and switching to another user with compromised credentials, your immediate thought should be "what privileges do I now have in this user context?" The answer? A sudo command for the `make` binary:

```
```