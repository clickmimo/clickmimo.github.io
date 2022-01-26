---
layout: post
permalink: /posts/thm/crack-the-hash
title: "Crack the hash"
---

Today we are going to crack some hashes. We will need a software to be able to identify different type of jashes. I'm going to use that one `https://gitlab.com/kalilinux/packages/hash-identifier/`. You can download it with the following command

```
wget https://gitlab.com/kalilinux/packages/hash-identifier/-/raw/kali/master/hash-id.py
```

Okay. Let's start with the challenge. Here is the first hash `48bb6e862e54f2a795ffc4e541caed4d`. When execute the python script it will prompt you to provide the hash that you want to identify.

```
python3 hash-id.py
```

![hash1](/assets/images/thm/crack-the-hash/hash1.png)

It is `md5`. Now we are going to use hashcat to crack it. From here `https://hashcat.net/wiki/doku.php?id=example_hashes` you can see the hash mode 

![hash-mode](/assets/images/thm/crack-the-hash/hash-mode.png)

Our hash is `md5` type, so we have to use hash-mode 0

```
hashcat -m 0 "48bb6e862e54f2a795ffc4e541caed4d" --force rockyou.txt 
```

![cracked](/assets/images/thm/crack-the-hash/cracked.png)

Good. We were able to crack it. The next one is `CBFDAC6008F9CAB4083784CBD1874F76618D2A97`. Paste the hash into the hash identifier.

![hash2](/assets/images/thm/crack-the-hash/hash2.png)

It is `SHA-1` type again we also need the hash mode.

![hash-mode2](/assets/images/thm/crack-the-hash/hash-mode2.png)

Use the following command to crack it.

```
hashcat -m 100 "CBFDAC6008F9CAB4083784CBD1874F76618D2A97" --force rockyou.txt
```

![cracked2](/assets/images/thm/crack-the-hash/cracked2.png)

The third hash is '1C8BFE8F801D79745C4631D09FFF36C82AA37FC4CCE4FC946683D7B336B63032'. Unfortunately our hash identifier wasn't able to find this hash. We can try with an online identifier `https://hashes.com`

![online-hash](/assets/images/thm/crack-the-hash/online-hash.png)

Next steps are the same find the hash mode and use hashcat to crack the hash.

```
hashcat -m 1400 "1C8BFE8F801D79745C4631D09FFF36C82AA37FC4CCE4FC946683D7B336B63032" --force rockyou.txt 
```

Hash number 4: `$2y$12$Dwt1BZj6pcyc3Dy1FWZ5ieeUznr71EeNkJkUlypTsgbX1H68wsRom` <br/>

![bcrypt-id](/assets/images/thm/crack-the-hash/bcrypt-id.png)

Here is the type and the hash mode is 3200. But that is not enough I faced some problems with this hash.

![error](/assets/images/thm/crack-the-hash/error.png)

After some research I find that I have to add backwardslashes before $ symbol `\$2y\$12\$Dwt1BZj6pcyc3Dy1FWZ5ieeUznr71EeNkJkUlypTsgbX1H68wsRom`. There is one more problem, if we execute hashcat with rockyou it will take us very long time. So I have made a new wordlist from rockyou, I took only the words that are under 5 symbols.

```
awk 'length < 5' rockyou.txt > rockyou4.txt
```

This command will take the words which are less than 5 symbols and it will write it into new wordlist. Now we are good to execute the hashcat.

```
hashcat -m 3200 hash.txt rockyou4.txt 
```

![cracked-bcrypt](/assets/images/thm/crack-the-hash/cracked-bcrypt.png)

Hash number 5: `279412f945939ba78ce0758d3fd83daa`

![md4-id](/assets/images/thm/crack-the-hash/md4-id.png)

```
hashcat -m 900 "279412f945939ba78ce0758d3fd83daa" rockyou.txt 
```

![md4-hashcat](/assets/images/thm/crack-the-hash/md4-hashcat.png)

We weren't able to crack it, probably the password is not in the rockyou file. Let's try to use an online tool like `https://crackstation.net/`

![crackstation](/assets/images/thm/crack-the-hash/crackstation.png)

At this moment I realized that `https://hashes.com` has already cracked the hash :D :D sometimes I'm not able to see the answer even if it infront of me. Whatever, we are done with the first part of the challenge.

Task 2 Hash 1: `F09EDCB1FCEFC6DFB23DC3505A882655FF77375ED8AA2D1C13F640FCCC2D0C85`

This time I'm going to use directly `https://crackstation.net/`

![crackstation-hash1](/assets/images/thm/crack-the-hash/crackstation-hash1.png)

Hash 2: `1DFECA0C002AE40B8619ECF94819CC1B`

![crackstation-hash2](/assets/images/thm/crack-the-hash/crackstation-hash2.png)

Again crackstation was good enough to crack the password.

Hash 3: `$6$aReallyHardSalt$6WKUTqzq.UQQmrm0p/T7MPpMbGNnzXPMAXi4bJMl9be.cfi3/qxIf.hsGpS41BqMhSrHVXgMpdjS6xeKZAs02.`

This time we are not able to crack it with crackstation. Identify the hash type.

![sha512-id](/assets/images/thm/crack-the-hash/sha512-id.png)

Copy the hash into a text file and crack it with hashcat. <br/>

This scan also will take a lot of time. Let's again generate new wordlist with max length of 6

```
awk 'length < 7' rockyou.txt > rockyou6.txt
```

```
hashcat -m 1800 hash.txt rockyou6.txt
```

![cracked-4](/assets/images/thm/crack-the-hash/cracked-4.png)

Here we go. We have successfully cracked the password.

Hash 4: `e5d8870e5bdd26602cab8dbe07a942c8669e56d6`

![sha1-id](/assets/images/thm/crack-the-hash/sha1-id.png)

It's `sha1` but it is little different because we are provided with a "salt". Copy the hash and the salt in the following format `e5d8870e5bdd26602cab8dbe07a942c8669e56d6:tryhackme`

```
hashcat -m 110 hash.txt /home/marin/Downloads/rockyou.txt 
```

Unfortunately we weren't able to crack the hash. If you look at the hint you will find out that I used a wrong hash mode it should be `HMAC-SHA1`

```
hashcat -m 160 hash.txtrockyou.txt
``` 

![cracked-last](/assets/images/thm/crack-the-hash/cracked-last.png)

Here we go, we were able to crack all the passwords.
