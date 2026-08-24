+++
title = 'BoardLight'
description = 'Easy Linux'
hideMeta = true
+++

nmap scan
![BoardLight (Easy Linux)-3.png](BoardLight%20%28Easy%20Linux%29-3.png)


![BoardLight (Easy Linux).png](BoardLight%20%28Easy%20Linux%29.png)
add board.htb to /etc/hosts file

There is /index.php - so php site

![BoardLight (Easy Linux)-1.png](BoardLight%20%28Easy%20Linux%29-1.png)

![BoardLight (Easy Linux)-2.png](BoardLight%20%28Easy%20Linux%29-2.png)
got a bit stuck here - but lets do VHost enumeration
`ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt -u http://board.htb -H "Host: FUZZ.board.htb" -c -v -mc all -fl 518`
Find one - crm.board.htb - add to /etc/hosts file
![BoardLight (Easy Linux)-4.png](BoardLight%20%28Easy%20Linux%29-4.png)

admin:admin worked to login

Searching for exploits for this version of Dolibarr, there is one that comes up straight away
https://github.com/nikn0laty/Exploit-for-Dolibarr-17.0.0-CVE-2023-30253


![BoardLight (Easy Linux)-5.png](BoardLight%20%28Easy%20Linux%29-5.png)
At htdocs/conf/conf.php the database connection information is present - found this by searching "dolibarr database location" online
![BoardLight (Easy Linux)-6.png](BoardLight%20%28Easy%20Linux%29-6.png)

`dolibarrowner` : `serverfun2$2023!!`

![BoardLight (Easy Linux)-7.png](BoardLight%20%28Easy%20Linux%29-7.png)
login, pass_encoding, pass, pass_crypted, pass_temp, firstname
![BoardLight (Easy Linux)-8.png](BoardLight%20%28Easy%20Linux%29-8.png)
Didn't have to do all that - password for db owner is the same for larissa

![BoardLight (Easy Linux)-9.png](BoardLight%20%28Easy%20Linux%29-9.png)

