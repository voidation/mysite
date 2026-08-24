+++
title = 'Editorial'
description = 'Easy Linux'
writeup = true
hideMeta = true
+++

nmap scan
![Editorial (Easy Linux).png](Editorial%20%28Easy%20Linux%29.png)

attack surface mainly looks like web app - add editorial.htb to /etc/hosts file

Hugo - 0.104.2 - static site generator

![Editorial (Easy Linux)-1.png](Editorial%20%28Easy%20Linux%29-1.png)

Add tiempoarriba.htb to /etc/hosts file and see whats on that site?

So /upload-cover makes an outbound http request based on the url provided
![Editorial (Easy Linux)-2.png](Editorial%20%28Easy%20Linux%29-2.png)

We can enumerate ports that may be open internally using this, as it seems that there is a much longer duration in the response coming back when accessing a url that actually is hosting something

The file upload doesn't seem to really do anything
The other site also seems to just redirect to the main site

![Editorial (Easy Linux)-3.png](Editorial%20%28Easy%20Linux%29-3.png)

port 5000 does not have a .jpeg extension - found by sorting for length

going to this new endpoint provided, it downloads a file containing API endpoint information in JSON format

![Editorial (Easy Linux)-4.png](Editorial%20%28Easy%20Linux%29-4.png)
![Editorial (Easy Linux)-5.png](Editorial%20%28Easy%20Linux%29-5.png)
`dev`:`dev080217_devAPI!@`
Attempt to ssh

![Editorial (Easy Linux)-6.png](Editorial%20%28Easy%20Linux%29-6.png)

There is a folder with .git directory - going through commits here
```bash
git log # commits and their messages and appropriate hashes
git show --name-status <HASH> # files that were modified during this commit
git show <HASH>:/path/to/file # display file content at that commit

```

![Editorial (Easy Linux)-7.png](Editorial%20%28Easy%20Linux%29-7.png)

`prod`:`080217_Producti0n_2023!@`

![Editorial (Easy Linux)-8.png](Editorial%20%28Easy%20Linux%29-8.png)

![Editorial (Easy Linux)-9.png](Editorial%20%28Easy%20Linux%29-9.png)
so from research, the protocol.ext.allow=always configuration here means that commands can be executed in the url provided

![Editorial (Easy Linux)-10.png](Editorial%20%28Easy%20Linux%29-10.png)

`sudo /usr/bin/python3 /opt/internal_apps/clone_changes/clone_prod_change.py 'ext::python3 /home/prod/shell.py'`


![Editorial (Easy Linux)-11.png](Editorial%20%28Easy%20Linux%29-11.png)
