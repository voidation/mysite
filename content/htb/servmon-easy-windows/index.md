+++
title = 'Servmon'
description = 'Easy Windows'
writeup = true
hideMeta = true
+++

run an nmap scan
![Servmon (Easy Windows).png](Servmon%20%28Easy%20Windows%29.png)
![Servmon (Easy Windows)-1.png](Servmon%20%28Easy%20Windows%29-1.png)
![Servmon (Easy Windows)-2.png](Servmon%20%28Easy%20Windows%29-2.png)
![Servmon (Easy Windows)-3.png](Servmon%20%28Easy%20Windows%29-3.png)

The first interesting port here is port 21 with ftp with anonymous login allowed.
we see 2 users - Nathan and Nadine
![Servmon (Easy Windows)-4.png](Servmon%20%28Easy%20Windows%29-4.png)
![Servmon (Easy Windows)-5.png](Servmon%20%28Easy%20Windows%29-5.png)
We see a message to from Nadine to Nathan that there is Passwords.txt in his Desktop
We also have Nathan's todo list
![Servmon (Easy Windows)-6.png](Servmon%20%28Easy%20Windows%29-6.png)

On port 80 there is a NVMS login portal - looking at it on burp, the login mechanism seems to be an api call in XML format:
![Servmon (Easy Windows)-7.png](Servmon%20%28Easy%20Windows%29-7.png)

NVMS 1000 is like some video surveillance thing and default creds are 'admin' '123456' - searching NVMS 1000 XXE in google actually leads to there being a directory traversal which is shown in exploitdb
https://www.exploit-db.com/exploits/47774

![Servmon (Easy Windows)-8.png](Servmon%20%28Easy%20Windows%29-8.png)

Able to fetch Users/Nathan/Passwords.txt
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
None of those passwords worked for NVMS (port 80) login with username 'admin' which was default or with 'nathan' and 'nadine'

lets try ssh
none of the passwords work for "nathan"

lets look at port 8443, which looks like https from the nmap output
visiting https://10.129.227.77:8443 we get the nsclient page

![Servmon (Easy Windows)-10.png](Servmon%20%28Easy%20Windows%29-10.png)

seems like the site is broken, lets try ssh with nadine, we get one to work!

`ssh nadine@10.129.227.77`
`L1k3B1gBut7s@W0rk`

![Servmon (Easy Windows)-11.png](Servmon%20%28Easy%20Windows%29-11.png)

lets see if nsclient works properly if we ssh tunnel to it (access it locally)

searching nsclient++ local privilege escalation, there is a CVE which mentions the storage of credentials in the nsclient.ini config file: https://nvd.nist.gov/vuln/detail/CVE-2025-34078
![Servmon (Easy Windows)-12.png](Servmon%20%28Easy%20Windows%29-12.png)

`ew2x6SsGTxjRwXOT`
![Servmon (Easy Windows)-13.png](Servmon%20%28Easy%20Windows%29-13.png)

port forward so we access it with the local machine and use the password

The web interface was shit, but essentially you had to go and make an external script and you had to call the key "Command" - this added the script in the queries tab which you could go to and see to run it:

![Servmon (Easy Windows)-14.png](Servmon%20%28Easy%20Windows%29-14.png)

putting in a reverse shell into c:\temp\shell.bat and having that as what is run by the query - we get a reverse shell as system

![Servmon (Easy Windows)-15.png](Servmon%20%28Easy%20Windows%29-15.png)

