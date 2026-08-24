+++
title = 'Magic'
description = 'Medium Linux'
writeup = true
hideMeta = true
+++

nmap scan.

![Magic (Medium Linux).png](Magic%20%28Medium%20Linux%29.png)

Only 22 and 80. The SSH version is old so there may be an exploit - there's a newer one but it requires a user already SSH'd into the box.

Connecting through the browser to http://magic.htb just hangs. Looking at the nmap scan, HEAD was the only method that worked? Let's fuzz for other paths.

VPN was busted - the home page loads fine now.

![Magic (Medium Linux)-1.png](Magic%20%28Medium%20Linux%29-1.png)

Bunch of images. `login.php`:

![Magic (Medium Linux)-2.png](Magic%20%28Medium%20Linux%29-2.png)

Typing a normal username pops the alert message:

![Magic (Medium Linux)-3.png](Magic%20%28Medium%20Linux%29-3.png)

But putting a quote in the username doesn't - possibly SQL injection? Same for the password.

So if it's something like a basic:

```sql
SELECT * FROM table WHERE username='' AND password='' LIMIT 1;
```

One quote and no alert, two quotes and there is an alert - suggests it's inside a SQL query somewhere.

Time for sqlmap.

![Magic (Medium Linux)-4.png](Magic%20%28Medium%20Linux%29-4.png)

It's an OR boolean-based blind SQLi. We can't simply bypass the login, so use sqlmap to enumerate the database. We could do it ourselves like this:

![Magic (Medium Linux)-5.png](Magic%20%28Medium%20Linux%29-5.png)

First letter of the database is 'M' - sqlmap does this automatically for us.

![Magic (Medium Linux)-6.png](Magic%20%28Medium%20Linux%29-6.png)

username = `admin`
password = `Th3s3usW4sK1ng`

![Magic (Medium Linux)-7.png](Magic%20%28Medium%20Linux%29-7.png)

Immediately thinking of uploading a PHP reverse shell.

![Magic (Medium Linux)-8.png](Magic%20%28Medium%20Linux%29-8.png)

Looks like there's some protection - let's try to bypass it.

![Magic (Medium Linux)-9.png](Magic%20%28Medium%20Linux%29-9.png)

Using PNG magic bytes; the only extension check was that `.png` had to be at the end.

![Magic (Medium Linux)-10.png](Magic%20%28Medium%20Linux%29-10.png)

Run the reverse shell.

![Magic (Medium Linux)-11.png](Magic%20%28Medium%20Linux%29-11.png)

Shell obtained!

![Magic (Medium Linux)-12.png](Magic%20%28Medium%20Linux%29-12.png)

The web app password had `Th3s3usW4sK1ng` - "theseus" in it - so I retried it with `su` and switched to that user.

![Magic (Medium Linux)-13.png](Magic%20%28Medium%20Linux%29-13.png)

We also have theseus's MySQL password:

![Magic (Medium Linux)-14.png](Magic%20%28Medium%20Linux%29-14.png)

`iamkingtheseus`

![Magic (Medium Linux)-15.png](Magic%20%28Medium%20Linux%29-15.png)

Ran the above to find binaries with the setuid bit:

![Magic (Medium Linux)-16.png](Magic%20%28Medium%20Linux%29-16.png)

Copilot says `sysinfo` is not a normal binary and likely custom. Running it just provides system info - output similar to `lshw`, which I figured out by pasting the output into Copilot.

We can modify `$PATH` in our session and place a malicious `lshw` binary, then call `sysinfo`.

So I compiled a reverse shell binary, placed it in `/home/theseus/bin`, and called it `lshw`.

Ran `/bin/sysinfo`:

![Magic (Medium Linux)-17.png](Magic%20%28Medium%20Linux%29-17.png)

Root shell.
