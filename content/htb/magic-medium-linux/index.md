+++
title = 'Magic'
description = 'Medium Linux'
writeup = true
hideMeta = true
+++

nmap scan
![Magic (Medium Linux).png](Magic%20%28Medium%20Linux%29.png)
Only 22 and 80 - SSH version is old so may be an exploit
Looked around there is a new exploit but it requires a user already sshed into the box

Connecting through the browser http://magic.htb just hangs. Looking at the nmap scan, it seems that HEAD was the only option that worked?

Let's fuzz for other paths

Okay so vpn was busted, you can actually load the home page
![Magic (Medium Linux)-1.png](Magic%20%28Medium%20Linux%29-1.png)

Bunch of images

login.php
![Magic (Medium Linux)-2.png](Magic%20%28Medium%20Linux%29-2.png)

Okay so typing a normal username in pops the alert message
![Magic (Medium Linux)-3.png](Magic%20%28Medium%20Linux%29-3.png)

But then putting a quote in the username doesn't? Possibly SQL injection? Same thing in password

so if its kinda like a basic
```sql
SELECT * FROM table WHERE username='' AND password='' LIMIT 1;
```

One quote and there's no alert, 2 quotes and there is an alert, which suggests somewhere in SQL query

Let us use sqlmap

![Magic (Medium Linux)-4.png](Magic%20%28Medium%20Linux%29-4.png)

It is an OR boolean-based blind SQLi - it seems that we can't simply bypass the login so using SQLmap we can enumerate the database. We could do this ourselves by:

![Magic (Medium Linux)-5.png](Magic%20%28Medium%20Linux%29-5.png)

First letter of database is 'M' - SQLmap is able to automatically do this for us.

![Magic (Medium Linux)-6.png](Magic%20%28Medium%20Linux%29-6.png)

username = `admin`
password = `Th3s3usW4sK1ng`

![Magic (Medium Linux)-7.png](Magic%20%28Medium%20Linux%29-7.png)

Immediately thinking of uploading a php reverse shell

![Magic (Medium Linux)-8.png](Magic%20%28Medium%20Linux%29-8.png)

Looks like there is some protection, lets try to bypass

![Magic (Medium Linux)-9.png](Magic%20%28Medium%20Linux%29-9.png)

Using Magic bytes for PNG and then the only check for extension was .png had to be at the end.
![Magic (Medium Linux)-10.png](Magic%20%28Medium%20Linux%29-10.png)

Lets run a reverse shell
![Magic (Medium Linux)-11.png](Magic%20%28Medium%20Linux%29-11.png)

shell obtained!

![Magic (Medium Linux)-12.png](Magic%20%28Medium%20Linux%29-12.png)
Considering the web app password had `Th3s3usW4sK1ng` theseus in it, I retried the password and switched to the user using su
![Magic (Medium Linux)-13.png](Magic%20%28Medium%20Linux%29-13.png)

We also have theseus password for mysql
![Magic (Medium Linux)-14.png](Magic%20%28Medium%20Linux%29-14.png)
`iamkingtheseus`

![Magic (Medium Linux)-15.png](Magic%20%28Medium%20Linux%29-15.png)

Ran the above to get binaries with setuid bit
![Magic (Medium Linux)-16.png](Magic%20%28Medium%20Linux%29-16.png)

Copilot says sysinfo is not a normal binary and likely custom

Running it just seems to provide systeminfo - it provides output similar to lshw which I kind of just found out by pasting the output into Copilot

We can modify $PATH in our session and place a malicious lshw binary and call sysinfo

So compiled a reverse shell binary and placed it in /home/theseus/bin and called it lshw

Ran /bin/sysinfo
![Magic (Medium Linux)-17.png](Magic%20%28Medium%20Linux%29-17.png)

