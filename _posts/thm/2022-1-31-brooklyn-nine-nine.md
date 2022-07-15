---
layout: post
permalink: /posts/thm/brooklyn-nine-nine
title: "Brooklyn Nine Nine"
---

This room is aimed for beginner level hackers but anyone can try to hack this box. <br/>

Let's first scan the target machine for open ports.

```
nmap -sC -sV <IP>
```

![nmap](/assets/images/thm/brooklyn-nine-nine/nmap.png)

There is an open port 21 FTP, 22 SSH and 80 HTTP. Let's try whether we are able to log in FTP server with anonymous access.

```
ftp <ip>
```

![ftp](/assets/images/thm/brooklyn-nine-nine/ftp.png)

We can download the file using the following command `get <FILE_NAME>`

![note-txt](/assets/images/thm/brooklyn-nine-nine/note-txt.png)

It looks that there is user with name jake which password is weak. Probably we'll try to brute force it later. Now go and visit the webpage.

![webpage](/assets/images/thm/brooklyn-nine-nine/webpage.png)

It's just an image. Let's view the source code.

![source](/assets/images/thm/brooklyn-nine-nine/source.png)

Very nice we have a commented clue, we are going to use some steganogtaphy which means there is something hidden behind the image. In addition we know the path to the image go to `<ip>/brooklyn99.jpg` and download the image.

![image](/assets/images/thm/brooklyn-nine-nine/image.png)

I tried to recevie some information with tools as exiftool, binwalk and stegehide. I didn't recevie any useful information from exiftool and binwalk. To be able to get any information with steghide we need a passphrase.

![passphrase](/assets/images/thm/brooklyn-nine-nine/passphrase.png)

So I decided to take an other way. As we mentioned earlier probably we need to try to brute force the jake password. I tried to brute force SSH service.

```
hydra -l jake -P /usr/share/wordlists/rockyou.txt 10.10.130.25 ssh
```

![hydra](/assets/images/thm/brooklyn-nine-nine/hydra.png)

Yes, we have found his ssh password. Let's get into it.

```
ssh jake@10.10.130.25
```

![jake-ssh](/assets/images/thm/brooklyn-nine-nine/jake-ssh.png)

Look around the system and you'll find the first flag. 

Let's try to escalate our privileges. Execute `sudo -l` to see whether our user is able to execute any commands as root.

![sudo-ls](/assets/images/thm/brooklyn-nine-nine/sudo-ls.png)

We can execute `less` as root. Go to <https://gtfobins.github.io/gtfobins/less/> and see how we can use it to get root access.

```
sudo less /etc/profile
!/bin/sh
```

![root](/assets/images/thm/brooklyn-nine-nine/root.png)

Here we go, we are root. Now just find the root flag. 

Okay we were able to find the both flags but I wasn't able to find out what is hidden inside the image. I tried all the passwords which I found as steghide passphrase but non of them was correct. After some research I found a tool called `stegcracker` which allowed me to find out the passphrase.

```
stegcracker brooklyn99.jpg /usr/share/wordlists/rockyou.txt
```

![stegcracker](/assets/images/thm/brooklyn-nine-nine/stegcracker.png)

It created a file 'brooklyn99.jpg.out'

![cat](/assets/images/thm/brooklyn-nine-nine/cat.png)

Well done. We have found one more password. That wasn't needed to complete the challenge but it is always good to learn something new.
