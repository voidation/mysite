+++
title = 'Writeup'
description = 'Easy Linux'
hideMeta = true
+++

- Fuzz directories?
- CMS Made Simple - docs - RCE says to login as admin - remote code exec? - content management system
- Look for other exploits

- Copyright @2019 - vulnerabilities in CMS 2019 versions
- searchsploit
[+] Salt for password found: 5a599ef579066807
[+] Username found: jkr
[+] Email found: jkr@writeup.htb
[+] Password found: 62def4866937f08cc13bab43bb14e6f7

jkr raykayjay9

/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games

user groups

/usr/local/bin - staff group has write and exec - no read

2025/08/05 05:51:01 CMD: UID=0     PID=30893  | /usr/sbin/CRON 
2025/08/05 05:51:01 CMD: UID=0     PID=30894  | /usr/sbin/CRON 
2025/08/05 05:51:01 CMD: UID=0     PID=30895  | /bin/sh -c /root/bin/cleanup.pl >/dev/null 2>&1 
     
2025/08/05 05:52:01 CMD: UID=0     PID=30896  | /usr/sbin/CRON 
2025/08/05 05:52:01 CMD: UID=0     PID=30897  | /usr/sbin/CRON 
2025/08/05 05:52:01 CMD: UID=0     PID=30898  | /bin/sh -c /root/bin/cleanup.pl >/dev/null 2>&1

seems to be removing anything in /usr/local/bin

peaspy




