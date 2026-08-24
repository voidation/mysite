+++
title = 'Snapped'
description = 'Hard Linux'
writeup = true
hideMeta = true
+++

nmap scan
![Snapped (Hard Linux).png](Snapped%20%28Hard%20Linux%29.png)
website, snapped.htb
![Snapped (Hard Linux)-1.png](Snapped%20%28Hard%20Linux%29-1.png)

Website static, fuzzing
![Snapped (Hard Linux)-5.png](Snapped%20%28Hard%20Linux%29-5.png)

subdomain enum - admin.snapped.htb
![Snapped (Hard Linux)-2.png](Snapped%20%28Hard%20Linux%29-2.png)

add to /etc/hosts and navigate reveals nginx ui page
![Snapped (Hard Linux)-3.png](Snapped%20%28Hard%20Linux%29-3.png)

nothing in exploitdb
![Snapped (Hard Linux)-4.png](Snapped%20%28Hard%20Linux%29-4.png)

fuzzing this page
![Snapped (Hard Linux)-9.png](Snapped%20%28Hard%20Linux%29-9.png)

/assets has directory listing
![Snapped (Hard Linux)-6.png](Snapped%20%28Hard%20Linux%29-6.png)

version files reveal possibly the nginx ui version?
![Snapped (Hard Linux)-7.png](Snapped%20%28Hard%20Linux%29-7.png)

![Snapped (Hard Linux)-8.png](Snapped%20%28Hard%20Linux%29-8.png)

Found this CVE after some searching

![Snapped (Hard Linux)-10.png](Snapped%20%28Hard%20Linux%29-10.png)

CVE information says `/api/backup` endpoint, navigating to this endpoint downloads a zip file

![Snapped (Hard Linux)-11.png](Snapped%20%28Hard%20Linux%29-11.png)
![Snapped (Hard Linux)-12.png](Snapped%20%28Hard%20Linux%29-12.png)

CVE details:
![Snapped (Hard Linux)-13.png](Snapped%20%28Hard%20Linux%29-13.png)

![Snapped (Hard Linux)-14.png](Snapped%20%28Hard%20Linux%29-14.png)

Article states AES-256
There's also a POC in the github page
https://github.com/0xJacky/nginx-ui/security/advisories/GHSA-g9w5-qffc-6762

![Snapped (Hard Linux)-15.png](Snapped%20%28Hard%20Linux%29-15.png)

decrypted successfully!!

![Snapped (Hard Linux)-16.png](Snapped%20%28Hard%20Linux%29-16.png)

database.db - has password hashes
![Snapped (Hard Linux)-17.png](Snapped%20%28Hard%20Linux%29-17.png)
`$2a$10$8YdBq4e.WeQn8gv9E0ehh.quy8D/4mXHHY4ALLMAzgFPTrIVltEvm`
`$2a$10$8M7JZSRLKdtJpx9YRUNTmODN.pKoBsoGCBi5Z8/WVGO2od9oCSyWq`

hashcat taking super long so checked to make sure this was the path, it is, just slow computer
hashcat cracks admin hash to `linkinpark`

password works for user jonathan with ssh

![Snapped (Hard Linux)-18.png](Snapped%20%28Hard%20Linux%29-18.png)

ran linpeas, nothing too obvious
 - no unusual suid files
 - no abusable capabilities
 - no extra groups
 - no obvious scheduled tasks

In jonathan's home directory, most suspicious thing is a snap directory with lots of files + config
![Snapped (Hard Linux)-19.png](Snapped%20%28Hard%20Linux%29-19.png)

checking snap version

![Snapped (Hard Linux)-20.png](Snapped%20%28Hard%20Linux%29-20.png)

![Snapped (Hard Linux)-21.png](Snapped%20%28Hard%20Linux%29-21.png)

Following https://github.com/TheCyberGeek/CVE-2026-3888-snap-confine-systemd-tmpfiles-LPE
compile and transfer
![Snapped (Hard Linux)-22.png](Snapped%20%28Hard%20Linux%29-22.png)

Not working, errors
![Snapped (Hard Linux)-23.png](Snapped%20%28Hard%20Linux%29-23.png)

Looking back through linpeas output

![Snapped (Hard Linux)-24.png](Snapped%20%28Hard%20Linux%29-24.png)

https://github.com/JuanBindez/CVE-2026-31431/blob/main/main.py

Use python exploit, got root shell!

![Snapped (Hard Linux)-25.png](Snapped%20%28Hard%20Linux%29-25.png)




