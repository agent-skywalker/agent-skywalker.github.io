https://online.pwntilldawn.com/Achievements/Details/11

![image](https://agent-skywalker.github.io/post/ptd/images/juno1.png)
Full Nmap scan 
```
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET POST OPTIONS HEAD
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Apache2 Debian Default Page: It works

NSE: Script Post-scanning.
Initiating NSE at 20:22
Completed NSE at 20:22, 0.00s elapsed
Initiating NSE at 20:22
Completed NSE at 20:22, 0.00s elapsed
Initiating NSE at 20:22
Completed NSE at 20:22, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 1379.37 seconds

```

The nmap scan indicates there's a web server running on port 80, so navigating to port 80 is the default apache page

![image](https://agent-skywalker.github.io/post/ptd/images/juno2.png)

Doing some files and directory brute-forcing i found `login.php` 

![image](https://agent-skywalker.github.io/post/ptd/images/juno3.png)

I navigated to the endpoint

![image](https://agent-skywalker.github.io/post/ptd/images/juno4.png)

But i don't have a pin and couldn't bypass it because there's no additional information about how the pin is.

So looking around, at the bottom of the page there's a link to download the android apk file

![image](https://agent-skywalker.github.io/post/ptd/images/juno13.png)

So i downloaded the apk file and tried to reverse engineer it using apktool but didn't get anything

So doing some more research i found this Mobile security Framework found [here](https://github.com/MobSF/Mobile-Security-Framework-MobSF) , cloned the repo and ran the setup script

```
./setup.sh
```

After it was done setting things up i launched the framework

```
./run.sh 127.0.0.1:9999
```

Specifying the interface and port for the framework to use

I then logged into the Mobile Security Framework web interface using default credentials  `mobsf/mobsf`

![image](https://agent-skywalker.github.io/post/ptd/images/juno5.png)

I uploaded the APK file onto the webpage to start the analysis.

![image](https://agent-skywalker.github.io/post/ptd/images/juno6.png)

Scrolling down to the bottom the page i was able to see all the strings extracted from the application

![image](https://agent-skywalker.github.io/post/ptd/images/juno7.png)

![image](https://agent-skywalker.github.io/post/ptd/images/juno8.png)

Reading through the strings i found FLAG43


![image](https://agent-skywalker.github.io/post/ptd/images/juno9.png)

Scrolling further downward i found the login pin for the website

![image](https://agent-skywalker.github.io/post/ptd/images/juno10.png)

I was able to log in and found the remaining 2 flags. 1 flag was Encoded and the other wasn't encoded

![image](https://agent-skywalker.github.io/post/ptd/images/juno11.png)

I worked out that the ASCII shift cypher is being used so we can decompile it.  
Then grab the last flag from the box!

![image](https://agent-skywalker.github.io/post/ptd/images/juno12.png)

