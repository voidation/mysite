+++
title = 'Editorial'
description = 'Easy Linux'
writeup = true
hideMeta = true
+++

nmap scan.

![Editorial (Easy Linux).png](Editorial%20%28Easy%20Linux%29.png)

The attack surface is mainly the web app - added `editorial.htb` to `/etc/hosts`.

It's Hugo 0.104.2, a static site generator.

![Editorial (Easy Linux)-1.png](Editorial%20%28Easy%20Linux%29-1.png)

Also added `tiempoarriba.htb` to `/etc/hosts` to see what's on that site.

`/upload-cover` makes an outbound HTTP request based on the URL provided.

![Editorial (Easy Linux)-2.png](Editorial%20%28Easy%20Linux%29-2.png)

We can enumerate internally open ports using this - responses take much longer when the URL actually hits something that's hosting a service.

The file upload doesn't seem to really do anything, and the other site just redirects to the main site.

![Editorial (Easy Linux)-3.png](Editorial%20%28Easy%20Linux%29-3.png)

Port 5000 doesn't return a `.jpeg` extension - found by sorting responses by length.

Hitting that endpoint downloads a file with API endpoint information in JSON format.

![Editorial (Easy Linux)-4.png](Editorial%20%28Easy%20Linux%29-4.png)
![Editorial (Easy Linux)-5.png](Editorial%20%28Easy%20Linux%29-5.png)

`dev`:`dev080217_devAPI!@`

Attempt SSH.

![Editorial (Easy Linux)-6.png](Editorial%20%28Easy%20Linux%29-6.png)

There's a folder with a `.git` directory - going through the commits:

```bash
git log # commits and their messages and appropriate hashes
git show --name-status <HASH> # files that were modified during this commit
git show <HASH>:/path/to/file # display file content at that commit
```

![Editorial (Easy Linux)-7.png](Editorial%20%28Easy%20Linux%29-7.png)

`prod`:`080217_Producti0n_2023!@`

![Editorial (Easy Linux)-8.png](Editorial%20%28Easy%20Linux%29-8.png)

![Editorial (Easy Linux)-9.png](Editorial%20%28Easy%20Linux%29-9.png)

From research, the `protocol.ext.allow=always` configuration means commands can be executed in the URL provided.

![Editorial (Easy Linux)-10.png](Editorial%20%28Easy%20Linux%29-10.png)

```bash
sudo /usr/bin/python3 /opt/internal_apps/clone_changes/clone_prod_change.py 'ext::python3 /home/prod/shell.py'
```

![Editorial (Easy Linux)-11.png](Editorial%20%28Easy%20Linux%29-11.png)

Root shell.
