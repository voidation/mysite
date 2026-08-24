+++
title = 'Keeper'
description = 'Easy Linux'
writeup = true
hideMeta = true
+++

nmap scan - port 80 and port 22 (SSH).

![Keeper (Easy Linux).png](Keeper%20%28Easy%20Linux%29.png)
![Keeper (Easy Linux)-1.png](Keeper%20%28Easy%20Linux%29-1.png)

Added `keeper.htb` and `tickets.keeper.htb` to `/etc/hosts`. Port 80 is a login page for a product called "Request Tracker".

![Keeper (Easy Linux)-2.png](Keeper%20%28Easy%20Linux%29-2.png)

Looking for default credentials in the setup docs.

![Keeper (Easy Linux)-3.png](Keeper%20%28Easy%20Linux%29-3.png)

They work!

Testing the search field - a single `'` does nothing but `''` shows results, which suggests SQLi.

Looking around the app:

![Keeper (Easy Linux)-4.png](Keeper%20%28Easy%20Linux%29-4.png)

Seems like `lnorgaard` is a new user with password `Welcome2023!`.

Tried it on SSH:

![Keeper (Easy Linux)-5.png](Keeper%20%28Easy%20Linux%29-5.png)
![Keeper (Easy Linux)-6.png](Keeper%20%28Easy%20Linux%29-6.png)

Downloaded this tool and followed the instructions to recover a passcode from the Keepass crash dump files sitting in the home directory:

https://github.com/JorianWoltjer/keepass-dump-extractor

![Keeper (Easy Linux)-7.png](Keeper%20%28Easy%20Linux%29-7.png)

`rødgrød med fløde`

That's the master key for the Keepass database. On Linux we can use the `keepassxc` CLI:

```bash
sudo apt install keepassxc
keepassxc open ./passcodes.kdbx
```

![Keeper (Easy Linux)-8.png](Keeper%20%28Easy%20Linux%29-8.png)
![Keeper (Easy Linux)-9.png](Keeper%20%28Easy%20Linux%29-9.png)

`F4><3K0nd!` - but that password doesn't seem to work for root.

The notes also contained a PuTTY private key:

```
PuTTY-User-Key-File-3: ssh-rsa
Encryption: none
Comment: rsa-key-20230519
Public-Lines: 6
AAAAB3NzaC1yc2EAAAADAQABAAABAQCnVqse/hMswGBRQsPsC/EwyxJvc8Wpul/D
8riCZV30ZbfEF09z0PNUn4DisesKB4x1KtqH0l8vPtRRiEzsBbn+mCpBLHBQ+81T
EHTc3ChyRYxk899PKSSqKDxUTZeFJ4FBAXqIxoJdpLHIMvh7ZyJNAy34lfcFC+LM
Cj/c6tQa2IaFfqcVJ+2bnR6UrUVRB4thmJca29JAq2p9BkdDGsiH8F8eanIBA1Tu
FVbUt2CenSUPDUAw7wIL56qC28w6q/qhm2LGOxXup6+LOjxGNNtA2zJ38P1FTfZQ
LxFVTWUKT8u8junnLk0kfnM4+bJ8g7MXLqbrtsgr5ywF6Ccxs0Et
Private-Lines: 14
AAABAQCB0dgBvETt8/UFNdG/X2hnXTPZKSzQxxkicDw6VR+1ye/t/dOS2yjbnr6j
oDni1wZdo7hTpJ5ZjdmzwxVCChNIc45cb3hXK3IYHe07psTuGgyYCSZWSGn8ZCih
kmyZTZOV9eq1D6P1uB6AXSKuwc03h97zOoyf6p+xgcYXwkp44/otK4ScF2hEputY
f7n24kvL0WlBQThsiLkKcz3/Cz7BdCkn+Lvf8iyA6VF0p14cFTM9Lsd7t/plLJzT
VkCew1DZuYnYOGQxHYW6WQ4V6rCwpsMSMLD450XJ4zfGLN8aw5KO1/TccbTgWivz
UXjcCAviPpmSXB19UG8JlTpgORyhAAAAgQD2kfhSA+/ASrc04ZIVagCge1Qq8iWs
OxG8eoCMW8DhhbvL6YKAfEvj3xeahXexlVwUOcDXO7Ti0QSV2sUw7E71cvl/ExGz
in6qyp3R4yAaV7PiMtLTgBkqs4AA3rcJZpJb01AZB8TBK91QIZGOswi3/uYrIZ1r
SsGN1FbK/meH9QAAAIEArbz8aWansqPtE+6Ye8Nq3G2R1PYhp5yXpxiE89L87NIV
09ygQ7Aec+C24TOykiwyPaOBlmMe+Nyaxss/gc7o9TnHNPFJ5iRyiXagT4E2WEEa
xHhv1PDdSrE8tB9V8ox1kxBrxAvYIZgceHRFrwPrF823PeNWLC2BNwEId0G76VkA
AACAVWJoksugJOovtA27Bamd7NRPvIa4dsMaQeXckVh19/TF8oZMDuJoiGyq6faD
AF9Z7Oehlo1Qt7oqGr8cVLbOT8aLqqbcax9nSKE67n7I5zrfoGynLzYkd3cETnGy
NNkjMjrocfmxfkvuJ7smEFMg7ZywW7CBWKGozgz67tKz9Is=
Private-MAC: b0a0fd2edf4f0e557200121aa673732c9e76750739db05adc3ab65ec34c55cb0
```

Saved that as a `.ppk` (PuTTY key) and converted it to a standard OpenSSH key with puttygen:

```bash
sudo apt install putty-tools
puttygen sshkey.ppk -O private-openssh -o sshkey
chmod 600 sshkey
```

SSH in as root with the key - done.
