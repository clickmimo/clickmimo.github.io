---
layout: post
permalink: /posts/thm/ninja-skills
title: "Ninja Skills"
---

Practise your Linux skills and complete the challenges. <br />

We have some files and our goal will be to find specific information about them. Here are the files:

``` 
   8V2L
   bny0
   c4ZX
   D8B3
   FHl1
   oiMO
   PFbD
   rmfX
   SRSq
   uqyw
   v2Vb
   X1Uy
```

Let's get started.

![website](/assets/images/thm/ninja-skills/welcome.png)

Firstly I searched for all files one by one.

```
find / -type f -name <file-name> 2>/dev/null
```
where

```
/ - means the root directory
-type f - will search for type files
-name - match the file name
2>/dev/null - this will send all the error responds to the /dev/null
```

For additional infomarmation about the find command you can use `man find` in your terminal. <br />

Later I have found an other way to search for all files with a single command.

```
find / -type f -name 8V2L -o -name bny0 -o -name c4ZX -o -name D8B3 -o -name FHl1 -o -name oiM0 -o -name PFbD -o -name rmfX -o -name SRSq -o -name uqyw -o -name v2Vb -o -name X1Uy 2>/dev/null
```
Here are where all the files located.

```
/etc/8V2L

/mnt/c4ZX
/mnt/D8B3
/var/FHl1
/opt/oiMO
/opt/PFbD
/media/rmfX
/etc/ssh/SRSq
/var/log/uqyw
/home/v2Vb
/X1Uy
```

I wasn't able to find `bny0` file. It probably doesn't exist. How ever let's start with the questions. <br />

`Question 1` Which of the above files are owned by the best-group group(enter the answer separated by spaces in alphabetical order) <br />

To find a group owner of the files we can use the following command

```
find / -type f -group best-group 2>/dev/null
```
![website](/assets/images/thm/ninja-skills/Q1.png)

`Question 2` Which of these files contain an IP address? <br />

For this one we can use regex. I'm not that familiar with it, so I just google something like `find ip addresses in a file using grep` 

```
grep -E -o "([0-9]{1,3}[\.]){3}[0-9]{1,3}" /etc/8V2L /mnt/c4ZX /mnt/D8B3 /var/FHl1 /opt/oiMO /opt/PFbD /media/rmfX /etc/ssh/SRSq /var/log/uqyw /home/v2Vb /X1Uy
```

![website](/assets/images/thm/ninja-skills/Q2.png)

`Question 3` Which file has the SHA1 hash of 9d54da7584015647ba052173b84d45e8007eba94 <br />

There are various checksum algorithms as md5sum, sha256sum. We are going to use sha1sum.

```
sha1sum /etc/8V2L /mnt/c4ZX /mnt/D8B3 /var/FHl1 /opt/oiMO /opt/PFbD /media/rmfX /etc/ssh/SRSq /var/log/uqyw /home/v2Vb /X1Uy
```  

![website](/assets/images/thm/ninja-skills/Q3.png)

`Question 4` Which file contains 230 lines? <br />

To print the line count of a file we can use `wc -l` 

```
wc -l /etc/8V2L /mnt/c4ZX /mnt/D8B3 /var/FHl1 /opt/oiMO /opt/PFbD /media/rmfX /etc/ssh/SRSq /var/log/uqyw /home/v2Vb /X1Uy
```

![website](/assets/images/thm/ninja-skills/Q4.png)

`Question 5` Which file's owner has an ID of 502? <br />

We can use `ls` command with the flag `-n` like `-l`, but list numeric user and group IDs

```
ls -n /etc/8V2L /mnt/c4ZX /mnt/D8B3 /var/FHl1 /opt/oiMO /opt/PFbD /media/rmfX /etc/ssh/SRSq /var/log/uqyw /home/v2Vb /X1Uy
```

![website](/assets/images/thm/ninja-skills/Q5.png)

`Question 6` 

Which file is executable by everyone? <br />

We can verify whether one file is executable or not as we see its permission. We can do that with `ls` and the flag `-l`, but I'm going to use `-la` it is the same but also shows the hidden files (Just a good habit to use `-la` instead of `-l`). 

```
ls -la /etc/8V2L /mnt/c4ZX /mnt/D8B3 /var/FHl1 /opt/oiMO /opt/PFbD /media/rmfX /etc/ssh/SRSq /var/log/uqyw /home/v2Vb /X1Uy
```

![website](/assets/images/thm/ninja-skills/Q6.png)

Well, on the left hand side there is something like `-rwxrwxr-x`, this shows the permission. The dash shows that this is a file it also could be `d` for the directory. The next characters has the following permission:

```
r - read permission
w - write permission
x - execute permission
```

The first 3 is for user, the next is for group and the last one is for others.
