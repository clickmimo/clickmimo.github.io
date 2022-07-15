---
layout: post
permalink: /posts/thm/bounty-hacker
title: "Bounty Hacker"
---

You talked a big game about being the most elite hacker in the solar system. Prove it and claim your right to the status of Elite Bounty Hacker! <br/>

Okay, let's start our enumeration with nmap scan

```
nmap -sC -sV <IP>
```

![nmap](/assets/images/thm/bounty-hacker/nmap.png)

Good! From the nmap scan we can see there are open ports 21 FTP, 22 SSH and 80 HTTP. Now we need to find a task list which will probably be located on the FTP server. You can notice from the nmap result that anonymous login is allowed. Go and visit the FTP server.

```
ftp <IP>
```

with username `anonymous`

![ftp](/assets/images/thm/bounty-hacker/ftp.png)

We can download the files with the following command

```
get <FILE_NAME>
```

From the `task.txt` we have found a name which is probably a user likewise `locks.txt` looks like password wordlist. Let's try to bruteforce ssh using hydra.

```
hydra -l lin -P locks.txt 10.10.126.63 ssh
```

![hydra](/assets/images/thm/bounty-hacker/hydra.png)

Yep, as we thought it was a password wordlist. Now SSH to the machine.

```
ssh lin@<IP>
```

![shell](/assets/images/thm/bounty-hacker/shell.png)

Here is the first flag. Now we need to find way to escalate our privileges. The first thing that I always check is whether the current user has any permissions to execute sudo commands.

```
sudo -l
```

![sudo-l](/assets/images/thm/bounty-hacker/sudo-l.png)

We are in luck. Lin user has the right to execute `/bin/tar` as sudo. Go and visit our favourite site <https://gtfobins.github.io/gtfobins/tar/>

![gtfobins](/assets/images/thm/bounty-hacker/gtfobins.png)

Yes, it is as simple as that, just copy the command and execute it and you will have root access.

```
sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
```

![root](/assets/images/thm/bounty-hacker/root.png)

Here we go, we are root. Find the root flag and you'll prove that you are an elite hacker.
