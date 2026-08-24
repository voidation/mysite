+++
title = 'Conversor'
description = 'Easy Linux'
hideMeta = true
+++

nmap scan
![Conversor (Easy Linux).png](Conversor%20%28Easy%20Linux%29.png)

go to web app runnin port 80 - 22 ssh version looks good

![Conversor (Easy Linux)-1.png](Conversor%20%28Easy%20Linux%29-1.png)

no checks on filename and saves file - arbritrary file upload
need some rce now

![Conversor (Easy Linux)-2.png](Conversor%20%28Easy%20Linux%29-2.png)

seems like they would have added this cron job cause there is a scripts folder present

runs every minute, lets put reverse shell in here

![Conversor (Easy Linux)-3.png](Conversor%20%28Easy%20Linux%29-3.png)

![Conversor (Easy Linux)-4.png](Conversor%20%28Easy%20Linux%29-4.png)

![Conversor (Easy Linux)-5.png](Conversor%20%28Easy%20Linux%29-5.png)

grab users.db
![Conversor (Easy Linux)-6.png](Conversor%20%28Easy%20Linux%29-6.png)
![Conversor (Easy Linux)-7.png](Conversor%20%28Easy%20Linux%29-7.png)
`fismathack`
`Keepmesafeandwarm`

login via ssh
![Conversor (Easy Linux)-8.png](Conversor%20%28Easy%20Linux%29-8.png)

https://github.com/pentestfunctions/CVE-2024-48990-PoC-Testing?tab=readme-ov-file
![Conversor (Easy Linux)-9.png](Conversor%20%28Easy%20Linux%29-9.png)

Definitely vulnerable to this CVE

![Conversor (Easy Linux)-10.png](Conversor%20%28Easy%20Linux%29-10.png)

https://github.com/ten-ops/CVE-2024-48990_needrestart/blob/main/makefile


