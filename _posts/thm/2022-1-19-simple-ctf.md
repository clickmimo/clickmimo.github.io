---
layout: post
permalink: /posts/thm/simple-ctf
title: "Simple CTF"
---

As always we are going to scan for open ports using nmap.

```
nmap -sC -sV <IP>
```

![nmap](/assets/images/thm/simple-ctf/nmap.png)

From our enumeration we can see there are open ports 21 FTP, 80 HTTP and 2222 which is unusual port for SSH. Let's first visit the website.

![website](/assets/images/thm/simple-ctf/website.png)

Hmm... it's the Apache default page. Probably there are some hidden directories. We can find out that with gobuster.

```
gobuster dir -u http://10.10.249.238/ -w /opt/SecLists/Discovery/Web-Content/directory-list-2.3-small.txt
```

Until we waiting for our web scan to complete, we can check whether we are able to find some information on the FTP server. We are going to try to get access with anonymous login (username: anonymous).

```
ftp 10.10.249.238
```

![ftp](/assets/images/thm/simple-ctf/ftp.png)

Yes, we were able to get into FTP.

![ftp-empty](/assets/images/thm/simple-ctf/ftp-empty.png)

But unfortunately there is nothing in it. Move back to the gobuster scan.

![gobuster](/assets/images/thm/simple-ctf/gobuster.png)

Well there is 'simple' directory, let's see what we can find there.

![simple](/assets/images/thm/simple-ctf/simple.png)

It's a Content Management System and probably there is an exploit for it. Scroll down to see the version.

![cms-version](/assets/images/thm/simple-ctf/cms-version.png)

Now just google it `cms made simple version 2.2.8 exploit`

![exploit-db](/assets/images/thm/simple-ctf/exploit-db.png)

Yes, we have found an existinct exploit based on python. Copy the script and run it. If you not sure how to use it, just run the script without any arguments(in my case it's `cms-made-simple-2-2-8.py`). 

```
python3 cms-made-simple-2-2-8.py
```

![syntax](/assets/images/thm/simple-ctf/syntax.png)

I got a syntaxis error so I went through the code and I added brackets on every print

![prints](/assets/images/thm/simple-ctf/prints.png)

NOTE! That is not the only place, look tha whole code. Once you are done run the script again.

![info](/assets/images/thm/simple-ctf/info.png)

We have to provide url and wordlist as following

```
python3 cms-made-simple-2-2-8.py -u http://10.10.249.238/simple/ --crack -w /opt/SecLists/Passwords/Common-Credentials/best110.txt
```

![crack-error](/assets/images/thm/simple-ctf/crack-error.png)

Hmm I got an error again. After some research and tests I found how we could change the script to work properly.

![code](/assets/images/thm/simple-ctf/code.png)

Save and run the script again

![cracked](/assets/images/thm/simple-ctf/cracked.png)

Finally we have found the password. <br />
Next question gives us a little clue. SSH to the machine

```
ssh mitch@10.10.249.238 -p 2222
```

![login](/assets/images/thm/simple-ctf/login.png)

Here we go, we are in. Look around the system to answer the next questions. <br />
Last thing is to escalate our privileges. Let's see if we have any sudo permissions with this user.

```
sudo -l
```

![sudo-l](/assets/images/thm/simple-ctf/sudo-l.png)

We are in luck, mitch user has the permssion to run vim with sudo rights. Go to `https://gtfobins.github.io/gtfobins/vim/#sudo` Just execute the first method and you'll get root access.

![root](/assets/images/thm/simple-ctf/root.png)

Here we go. Now just find the root flag.