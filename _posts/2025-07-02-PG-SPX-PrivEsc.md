---
layout: post
title: "Proving Grounds SPX: Privilege Escalation"
---

As I continue to progress through [TJ Null's OSCP PWK V3 list](https://docs.google.com/spreadsheets/u/1/d/1dwSMIAPIam0PuRBkCiDI88pU3yzrqqHkDtBngUHNCw8/htmlview), I wanted to cover a particular machine that had a very unique privilege escalation vulnerability. The machine in question is SPX. I'll skip over initial access for the sake of brevity and limiting spoilers.

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

I'll be honest, I was stumped on this problem for a good hour or so. I decided to thoroughly enumerate the rest of the system. I was convinced I missed something somewhere. Eventually, I realized the `sudo` permission for this new user was the obvious privilege escalation vulnerability. It was my only direct access to command execution as *root*.

### Even More Research

Having used the `make` utility in the past, I knew it calls on instructions specified in a Makefile. I searched for Makefile exploits/privilege escalation techniques. A few Google searches eventually led me to this [Medium](https://medium.com/@adamforsythebartlett/makefile-privilege-escalation-oscp-62ea2c666d23) article, written by one Adam Bartlett. He explains that `make` can be used to execute bash commands in the same directory that the `Makefile` resides in. Here is the malicious syntax for bash execution.

![Medium Article Example](/assets/img/PG-SPX-PrivEsc_Make-Medium-Blog.png)

# Crafting the Makefile

Now it's time to match the example to our target system. I always like to start by making a backup of a file I'm modifying. It's not a requirement by any means, but it's good practice.

```
profiler@spx:/var/www$ cd ~/php-spx/
profiler@spx:~/php-spx$ cp Makefile ./Makefile.bak
```

There's many different bash techniques that can be used to spawn a root shell. I chose to create a malicous copy of bash with an SUID bit. This allows any user to execute this bash copy with a `-p` flag, which tells bash to not reset the effective user ID with my real user ID. In short, bash will run with the file owner's permissions (root).

![Malicious Makefile](/assets/img/PG-SPX-PrivEsc_New-Makefile.png)

# Escalating Privileges

With the malicious Makefile in place, it's simply a matter of executing profiler's allowed `sudo` command.

![Sudo Make Exploit](/assets/img/PG-SPX-PrivEsc_Sudo-Make-Exploit.png)

Now I'll execute the new `/tmp/evil` bash copy, ensuring the `-p` flag is provided as an argument to spawn a root shell!

![Evil Bash Copy](/assets/img/PG-SPX-PrivEsc_Bash-Privilege-Escalation.png)

# Lessons Learned

The very specific `sudo` command is a great starting point, but the true vulnerability lies in the specified directory, `/home/profiler/php-spx`, where the standard user `profiler` has *write* access to the directory, and as a result, to the *Makefile* itself. If the specified directory prevented the `profiler` user write access, or was instead owned by `root`, there would be no opportunity to escalate privileges. In short, the devil is always in the details, and file permissions matter!

![Lessons Learned](/assets/img/PG-SPX-PrivEsc_Lessons-Learned.png)

