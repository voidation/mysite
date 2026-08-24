+++
title = 'Servmon'
description = 'Easy Windows'
writeup = true
hideMeta = true
+++

Ran an nmap scan.

![Servmon (Easy Windows).png](Servmon%20%28Easy%20Windows%29.png)
![Servmon (Easy Windows)-1.png](Servmon%20%28Easy%20Windows%29-1.png)
![Servmon (Easy Windows)-2.png](Servmon%20%28Easy%20Windows%29-2.png)
![Servmon (Easy Windows)-3.png](Servmon%20%28Easy%20Windows%29-3.png)

The first interesting port is 21 - FTP with anonymous login allowed. We see 2 users: Nathan and Nadine.

![Servmon (Easy Windows)-4.png](Servmon%20%28Easy%20Windows%29-4.png)
![Servmon (Easy Windows)-5.png](Servmon%20%28Easy%20Windows%29-5.png)

A message from Nadine to Nathan mentions Passwords.txt on his Desktop. We also have Nathan's todo list.

![Servmon (Easy Windows)-6.png](Servmon%20%28Easy%20Windows%29-6.png)

Port 80 has an NVMS login portal - looking at it in Burp, the login mechanism is an API call in XML format:

![Servmon (Easy Windows)-7.png](Servmon%20%28Easy%20Windows%29-7.png)

NVMS 1000 is a video surveillance thing; default creds are `admin`/`123456`. Searching "NVMS 1000 XXE" leads to a directory traversal shown on exploitdb:

https://www.exploit-db.com/exploits/47774

![Servmon (Easy Windows)-8.png](Servmon%20%28Easy%20Windows%29-8.png)

Able to fetch `Users/Nathan/Passwords.txt`:

![Servmon (Easy Windows)-9.png](Servmon%20%28Easy%20Windows%29-9.png)

```
1nsp3ctTh3Way2Mars!
Th3r34r3To0M4nyTrait0r5!
B3WithM30r4ga1n5tMe
L1k3B1gBut7s@W0rk
0nly7h3y0unGWi11F0l10w
IfH3s4b0Utg0t0H1sH0me
Gr4etN3w5w17hMySk1Pa5$
```

None of those passwords worked for the NVMS login on port 80 - not with username `admin` (the default) or `nathan`/`nadine`.

Let's try SSH. None work for "nathan" - but port 8443 looks like HTTPS from the nmap output. Visiting https://10.129.227.77:8443 gives the nsclient page:

![Servmon (Easy Windows)-10.png](Servmon%20%28Easy%20Windows%29-10.png)

The site looks broken - let's try SSH with nadine. One works:

```bash
ssh nadine@10.129.227.77
```

`L1k3B1gBut7s@W0rk`

![Servmon (Easy Windows)-11.png](Servmon%20%28Easy%20Windows%29-11.png)

Let's see if nsclient works properly when SSH-tunnelled (accessed locally). Searching "nsclient++ local privilege escalation" there's a CVE about credentials stored in the `nsclient.ini` config file: https://nvd.nist.gov/vuln/detail/CVE-2025-34078

![Servmon (Easy Windows)-12.png](Servmon%20%28Easy%20Windows%29-12.png)

`ew2x6SsGTxjRwXOT`

![Servmon (Easy Windows)-13.png](Servmon%20%28Easy%20Windows%29-13.png)

Port-forwarded so we access it with the local machine and use the password.

The web interface was rough, but essentially you create an external script and call the key "Command" - that adds the script to the queries tab, which you can open and run:

![Servmon (Easy Windows)-14.png](Servmon%20%28Easy%20Windows%29-14.png)

Put a reverse shell into `c:\temp\shell.bat` and have that be what the query runs - we get a reverse shell as SYSTEM.

![Servmon (Easy Windows)-15.png](Servmon%20%28Easy%20Windows%29-15.png)
