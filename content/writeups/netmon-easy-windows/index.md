+++
title = 'Netmon'
description = 'Easy Windows'
writeup = true
hideMeta = true
+++

![Netmon (Easy Windows).png](Netmon%20%28Easy%20Windows%29.png)
![Netmon (Easy Windows)-3.png](Netmon%20%28Easy%20Windows%29-3.png)

Port 80.

![Netmon (Easy Windows)-2.png](Netmon%20%28Easy%20Windows%29-2.png)

Looking at the FTP service, the whole C drive appears to be mounted to it. We can grab user.txt straight from the Public user's Desktop over FTP.

Port 80 runs the PRTG network monitoring tool - found its folder in Program Files (x86) and searched around for version/config files. I wasn't sure where to look at first, but did spot the Paessler stuff.

Checked Ippsec's video for hints - `dir -a` to show hidden files, and sure enough `ProgramData` was there.

Got the config file:

![Netmon (Easy Windows)-4.png](Netmon%20%28Easy%20Windows%29-4.png)

In `PRTG Configuration.old.bak`:

![Netmon (Easy Windows)-5.png](Netmon%20%28Easy%20Windows%29-5.png)

`prtgadmin`
`PrTg@dmin2018`

Didn't work - but what are the chances they just incremented the year? `PrTg@dmin2019` (the creation date was the giveaway). That works.

![Netmon (Easy Windows)-6.png](Netmon%20%28Easy%20Windows%29-6.png)

PRTG version = 18.1.37.13946.

![Netmon (Easy Windows)-7.png](Netmon%20%28Easy%20Windows%29-7.png)

Resources for the CVE:

https://sensepost.com/blog/2019/being-stubborn-pays-off-pt.-1-cve-2018-19204/
https://gist.github.com/n30m1nd/1788ab84b94a03c62847d285ee0cfe81

There's also CVE-2018-9276, for which I found this exploit.py:

https://github.com/A1vinSmith/CVE-2018-9276/blob/main/exploit.py

Downloaded and ran it with the right arguments - shell as System, which let us grab root.txt.

That exploit was essentially written because of the Netmon box, so I wanted to learn how to do it manually. Watched Ippsec's video and documented the manual steps below:

![Netmon (Easy Windows)-8.png](Netmon%20%28Easy%20Windows%29-8.png)
![Netmon (Easy Windows)-9.png](Netmon%20%28Easy%20Windows%29-9.png)

To execute it:

![Netmon (Easy Windows)-10.png](Netmon%20%28Easy%20Windows%29-10.png)
![Netmon (Easy Windows)-11.png](Netmon%20%28Easy%20Windows%29-11.png)
