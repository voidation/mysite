+++
title = 'Netmon'
description = 'Easy Windows'
writeup = true
hideMeta = true
+++

![Netmon (Easy Windows).png](Netmon%20%28Easy%20Windows%29.png)
![Netmon (Easy Windows)-3.png](Netmon%20%28Easy%20Windows%29-3.png)
port 80
![Netmon (Easy Windows)-2.png](Netmon%20%28Easy%20Windows%29-2.png)

So looking at the ftp service, it seems that the C drive has been mounted to it

We are able to get user.txt just from ftp service in the Public user Desktop

Seeing the PRTG network tool on port 80, we find the folder for that in Program Files x86 - I looked a lot here for something - I searched up where to find version/config files and didnt know what - did see the Paessler thing

looked at ippsec video for hints - `dir -a` to show hidden files and sure enough ProgramData was there

Get the config file
![Netmon (Easy Windows)-4.png](Netmon%20%28Easy%20Windows%29-4.png)

in PRTG Configuration.old.bak
![Netmon (Easy Windows)-5.png](Netmon%20%28Easy%20Windows%29-5.png)

`prtgadmin`
`PrTg@dmin2018`

bruh - "what are the chances that they incremented the year"
`PrTg@dmin2019` - cause of the creation date
that works....

![Netmon (Easy Windows)-6.png](Netmon%20%28Easy%20Windows%29-6.png)
prtg version = 18.1.37.13946

![Netmon (Easy Windows)-7.png](Netmon%20%28Easy%20Windows%29-7.png)
https://sensepost.com/blog/2019/being-stubborn-pays-off-pt.-1-cve-2018-19204/
https://gist.github.com/n30m1nd/1788ab84b94a03c62847d285ee0cfe81

Another vulnerability was CVE-2018-19204 for which I found this exploit.py

https://github.com/A1vinSmith/CVE-2018-9276/blob/main/exploit.py

Downloading this and running it with the correct arguments provided a shell as System
which allowed us to get the root.txt

However, this exploit was essentially created because of the Netmon box - so I want to learn how to do this exploit manually - for which I will watch Ippsec's video and do it myself and document below:

![Netmon (Easy Windows)-8.png](Netmon%20%28Easy%20Windows%29-8.png)
![Netmon (Easy Windows)-9.png](Netmon%20%28Easy%20Windows%29-9.png)
To execute it:
![Netmon (Easy Windows)-10.png](Netmon%20%28Easy%20Windows%29-10.png)
![Netmon (Easy Windows)-11.png](Netmon%20%28Easy%20Windows%29-11.png)

