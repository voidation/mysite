+++
title = 'Monitored'
description = 'Medium Linux'
hideMeta = true
+++

nmap
![Monitored (Medium Linux).png](Monitored%20%28Medium%20Linux%29.png)
![Monitored (Medium Linux)-1.png](Monitored%20%28Medium%20Linux%29-1.png)

SSH is 8.4p1 - checked if vulnerable to CVE-2024-6387 with https://github.com/Karmakstylez/CVE-2024-6387
Not vulnerable!

Lets move onto port 80
Redirects to https://nagios.monitored.htb - which is hosted at port 443
support@monitored.htb - is an account - found from part of SSL cert

![Monitored (Medium Linux)-2.png](Monitored%20%28Medium%20Linux%29-2.png)
Seems like the latest Nagios software

Nagios XI is an enterprise grade IT infrastructure monitoring solution

Port 5667 which is also open on the box is commonly associated with NSCA (Nagios Service Check Acceptor) for sending passive monitoring data to a Nagios server

Default credentials for logging in did not work

Other endpoints like /nagios/cgi-bin - comes up with HTTP basic auth

UDP ports open

![Monitored (Medium Linux)-3.png](Monitored%20%28Medium%20Linux%29-3.png)


`snmpwalk -c public -v2c 10.129.230.96 | tee output.snmp`

enumerating all the information provided via snmp

![Monitored (Medium Linux)-4.png](Monitored%20%28Medium%20Linux%29-4.png)
`svc`:`XjH7VCehowpR1xZB`

![Monitored (Medium Linux)-5.png](Monitored%20%28Medium%20Linux%29-5.png)

![Pasted image 20260115150752.png](Pasted%20image%2020260115150752.png)

![Monitored (Medium Linux)-7.png](Monitored%20%28Medium%20Linux%29-7.png)

Nagios xi version is 5.11.0

