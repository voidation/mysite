+++
title = 'Snapped'
description = 'Hard Linux'
writeup = true
hideMeta = true
+++

Kicked off with an nmap scan.

![Snapped (Hard Linux).png](Snapped%20%28Hard%20Linux%29.png)

The site is `snapped.htb`.

![Snapped (Hard Linux)-1.png](Snapped%20%28Hard%20Linux%29-1.png)

The website is fully static, so I started fuzzing.

![Snapped (Hard Linux)-5.png](Snapped%20%28Hard%20Linux%29-5.png)

Subdomain enumeration turned up `admin.snapped.htb`.

![Snapped (Hard Linux)-2.png](Snapped%20%28Hard%20Linux%29-2.png)

Added it to `/etc/hosts` and navigated there - it reveals an nginx UI page.

![Snapped (Hard Linux)-3.png](Snapped%20%28Hard%20Linux%29-3.png)

Nothing relevant in exploitdb.

![Snapped (Hard Linux)-4.png](Snapped%20%28Hard%20Linux%29-4.png)

Fuzzing the admin panel.

![Snapped (Hard Linux)-9.png](Snapped%20%28Hard%20Linux%29-9.png)

`/assets` has directory listing enabled.

![Snapped (Hard Linux)-6.png](Snapped%20%28Hard%20Linux%29-6.png)

Version files reveal the nginx UI version.

![Snapped (Hard Linux)-7.png](Snapped%20%28Hard%20Linux%29-7.png)

![Snapped (Hard Linux)-8.png](Snapped%20%28Hard%20Linux%29-8.png)

Found a CVE for this version after some searching.

![Snapped (Hard Linux)-10.png](Snapped%20%28Hard%20Linux%29-10.png)

The CVE details point at the `/api/backup` endpoint - navigating there downloads a zip file.

![Snapped (Hard Linux)-11.png](Snapped%20%28Hard%20Linux%29-11.png)
![Snapped (Hard Linux)-12.png](Snapped%20%28Hard%20Linux%29-12.png)

CVE details:

![Snapped (Hard Linux)-13.png](Snapped%20%28Hard%20Linux%29-13.png)

![Snapped (Hard Linux)-14.png](Snapped%20%28Hard%20Linux%29-14.png)

The article states the backup is encrypted with AES-256. There's also a PoC on the advisory's GitHub page:

https://github.com/0xJacky/nginx-ui/security/advisories/GHSA-g9w5-qffc-6762

![Snapped (Hard Linux)-15.png](Snapped%20%28Hard%20Linux%29-15.png)

Decrypted successfully!

![Snapped (Hard Linux)-16.png](Snapped%20%28Hard%20Linux%29-16.png)

`database.db` contains password hashes.

![Snapped (Hard Linux)-17.png](Snapped%20%28Hard%20Linux%29-17.png)
`$2a$10$8YdBq4e.WeQn8gv9E0ehh.quy8D/4mXHHY4ALLMAzgFPTrIVltEvm`
`$2a$10$8M7JZSRLKdtJpx9YRUNTmODN.pKoBsoGCBi5Z8/WVGO2od9oCSyWq`

Hashcat was taking ages on my machine, so I double-checked this was the right path - it was, just a slow computer. Eventually it cracked the admin hash: `linkinpark`.

That password works for user `jonathan` over SSH.

![Snapped (Hard Linux)-18.png](Snapped%20%28Hard%20Linux%29-18.png)

Ran linpeas - nothing too obvious:
- no unusual SUID files
- no abusable capabilities
- no extra groups
- no obvious scheduled tasks

In jonathan's home directory, the most suspicious thing is a `snap` directory with lots of files and config.

![Snapped (Hard Linux)-19.png](Snapped%20%28Hard%20Linux%29-19.png)

Checking the snap version.

![Snapped (Hard Linux)-20.png](Snapped%20%28Hard%20Linux%29-20.png)

![Snapped (Hard Linux)-21.png](Snapped%20%28Hard%20Linux%29-21.png)

Following the snap-confine / systemd tmpfiles LPE:

https://github.com/TheCyberGeek/CVE-2026-3888-snap-confine-systemd-tmpfiles-LPE

Compiled and transferred it over.

![Snapped (Hard Linux)-22.png](Snapped%20%28Hard%20Linux%29-22.png)

Not working - errors.

![Snapped (Hard Linux)-23.png](Snapped%20%28Hard%20Linux%29-23.png)

Looking back through the linpeas output.

![Snapped (Hard Linux)-24.png](Snapped%20%28Hard%20Linux%29-24.png)

https://github.com/JuanBindez/CVE-2026-31431/blob/main/main.py

Used the python exploit - got a root shell!

![Snapped (Hard Linux)-25.png](Snapped%20%28Hard%20Linux%29-25.png)
