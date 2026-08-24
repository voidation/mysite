+++
title = 'Sau'
description = 'Easy Linux'
writeup = true
hideMeta = true
+++

![Sau (Easy Linux).png](Sau%20%28Easy%20Linux%29.png)

Made a new basket on the request-baskets instance (v1.2.1).

There's a CVE for this version - a POST to a vulnerable endpoint makes the server itself request a locally running service. Useful, since port 80 is filtered externally in the nmap scan.

Pointed it at the local mailtrail service, which has its own simple CVE providing a reverse shell.

Privesc: `sudo -l` allowed `systemctl`, which pages output through `less` - escaped with `!sh` for a root shell.
