---
layout: post
title: "Proving Grounds SPX: Privilege Escalation"
---

As I continue to progress through [TJ Null's OSCP PWK V3 list](https://docs.google.com/spreadsheets/u/1/d/1dwSMIAPIam0PuRBkCiDI88pU3yzrqqHkDtBngUHNCw8/htmlview), I wanted to cover a particular machine that had a very unique privilege escalation vulnerability. This machine is given the name SPX. I'll skip over initial access for the sake of brevity and limiting spoilers.

Speaking of spoilers, if you plan on completing this machine and don't want answers given away, DO NOT PROCEED!

# Identifying the Vulnerability

After gaining your initial foothold, and switching to another user with compromised credentials, your immediate thought should be:

*what privileges do I have in this new user context?* 

The answer? A sudo command for the `make` binary:

![profiler sudo privileges](/assets/img/PG-SPX-PrivEsc_Sudo-Cmd.png)

## Research

Anytime you come across a specific binary with special permissions (`sudo` in this case), always consult the [GTFOBins](https://gtfobins.github.io/gtfobins/make/#sudo) GitHub Pages site.

Initially, it looks like we have an easy win here, but we do not have access to the `-s` and `--eval` flags shown in the example code snippet. Remember, the whitelisted command from the `sudo -l` output was `/usr/bin/make install -C /home/profiler/php-spx`:

![GTFOBin Research](/assets/img/PG-SPX-PrivEsc_Make-GTFOBin.png)

I'll be honest, I was stumped on this problem for a good hour or so. I decided to thoroughly enumerate the rest of the system. I was convinced I missed something somewhere. Eventually, I did realize that the `sudo` permission for this new user was the obvious privilege escalation vulnerability. It was my only direct access to command execution as *root*.

### Even More Research

