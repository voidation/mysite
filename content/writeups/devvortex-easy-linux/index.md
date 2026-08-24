+++
title = 'Devvortex'
description = 'Easy Linux'
writeup = true
hideMeta = true
+++

nmap scan:

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.9 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
```

Ports 80 and 22 (http and ssh). The site tries to redirect to http://devvortex.htb/ - changed `/etc/hosts`.

Explored - not too much to interact with.

![Devvortex (Easy Linux).png](Devvortex%20%28Easy%20Linux%29.png)

There are a couple of inputs for email and name to get a "call back" - doesn't seem to have injection. `/images/xxx.png` - a few images exist like this in the source HTML - don't think that's a vuln.

Running out of ideas - time to fuzz. While the fuzz ran, checked versions for SSH. OpenSSH 8.2p1 looked potentially vulnerable.

![Devvortex (Easy Linux)-1.png](Devvortex%20%28Easy%20Linux%29-1.png)
![Devvortex (Easy Linux)-2.png](Devvortex%20%28Easy%20Linux%29-2.png)

Not vulnerable to Terrapin.

Looked at other things - lost. Did a couple of questions on the guided mode of HackTheBox - **I forgot to fuzz subdomains**. That's it for today, pick it up tomorrow - next step is fuzz subdomains.

![Devvortex (Easy Linux)-3.png](Devvortex%20%28Easy%20Linux%29-3.png)

Fuzzed and found `dev.devvortex.htb`. Based on the HTTP response, the site uses the Cassiopeia CMS.

![Devvortex (Easy Linux)-4.png](Devvortex%20%28Easy%20Linux%29-4.png)

Interesting page? Can see `/index.php` - so it's a PHP server. The site is using Joomla.

Searching for Joomla vulnerabilities - Joomla! 4 exposes version information in `/language/en-GB/langmetadata.xml`.

![Devvortex (Easy Linux)-5.png](Devvortex%20%28Easy%20Linux%29-5.png)

Potential exploits:

![Devvortex (Easy Linux)-6.png](Devvortex%20%28Easy%20Linux%29-6.png)
![Devvortex (Easy Linux)-7.png](Devvortex%20%28Easy%20Linux%29-7.png)

Login page exposed. Searching "joomla 4.2.6 administrator login vulnerability" → https://www.vulncheck.com/blog/joomla-for-rce → CVE-2023-23752.

```bash
curl -v http://dev.devvortex.htb/api/index.php/v1/config/application?public=true
```

Provides:

![Devvortex (Easy Linux)-8.png](Devvortex%20%28Easy%20Linux%29-8.png)

- user: `lewis`
- password: `P4ntherg0t1n5r3c0n##`

Logged into the Joomla admin portal. Following the path to code execution: System → Administrator Templates → Atum → Details and Files.

![Devvortex (Easy Linux)-9.png](Devvortex%20%28Easy%20Linux%29-9.png)

Added a simple line to `index.php` to get command execution. Stole the PHP reverse shell from https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php

![Devvortex (Easy Linux)-10.png](Devvortex%20%28Easy%20Linux%29-10.png)

As www-data - checking `/etc/passwd`, the interesting user is `logan` (`/home/logan:/bin/bash`).

Got the reverse shell - now to privesc. Attack path: look for logan's password somewhere on the box. Run linpeas, run find commands in suspicious locations.

Using `ss -tuln` / `ss -tulp` we can see MySQL is running on the box, and with lewis's creds we can log in:

![Devvortex (Easy Linux)-11.png](Devvortex%20%28Easy%20Linux%29-11.png)

Listed the databases - the Joomla one has the standard `sd4fg_*` table set (users, etc.).

![Devvortex (Easy Linux)-12.png](Devvortex%20%28Easy%20Linux%29-12.png)

`logan@devvortex.htb`
`$2y$10$IT4k5kmSGvHSO9d6M/1w0eYiB5Ne9XzArQRFJTGThNiy/yBtkIj12`

```bash
hashcat -a 0 -m 3200 hash.txt /usr/share/wordlists/rockyou.txt
```

Cracked! Password for logan is `tequieromucho`.

`logan@devvortex.htb` / `tequieromucho`

SSH in as logan - user pwned!

![Devvortex (Easy Linux)-13.png](Devvortex%20%28Easy%20Linux%29-13.png)

We can run `apport-cli` as root - version 2.20.11. Searching this up, there's CVE-2023-1326 which leads to privilege escalation.

Using the help page, running

```bash
/usr/bin/apport-cli -P 0 -f --save=/home/logan/a.crash
```

creates a valid crash file, which we can then do

```bash
sudo /usr/bin/apport-cli -c a.crash
```

and "view" - that drops us into a `less` view of the report, which we can exit with `!/bin/bash` giving us root.
