![image](https://agent-skywalker.github.io/post/ptd/images/snare1.png)

# Reconnaissance 
Full nmap scan

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 2f:0e:73:d4:ae:73:14:7e:c5:1c:15:84:ef:45:a4:d1 (RSA)
|   256 39:0b:0b:c9:86:c9:8e:b5:2b:0c:39:c7:63:ec:e2:10 (ECDSA)
|_  256 f6:bf:c5:03:5b:df:e5:e1:f4:da:ac:1e:b2:07:88:2f (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
| http-title: Welcome to my homepage!
|_Requested resource was /index.php?page=home
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

NSE: Script Post-scanning.
Initiating NSE at 03:30
Completed NSE at 03:30, 0.00s elapsed
Initiating NSE at 03:30
Completed NSE at 03:30, 0.00s elapsed
Initiating NSE at 03:30
Completed NSE at 03:30, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 2156.22 seconds
```

It shows there's a web server running on port 80

Navigating to Port 80 is this Webpage

![image](https://agent-skywalker.github.io/post/ptd/images/snare2.png)

Studying the url it looks it is vulnerable to remote file inclusion vulnerability because the `page` parameter is being used to include a file dynamically (where a web application includes a file based on user input, without properly validating or sanitizing that input.), which can be controlled by an attacker. In this case, the `page` parameter is set to `home`, which suggests that the script is intended to include a file named `home.php` or `home.inc` or something similar.

So to test it out we setup a python http server and try to get a file that doesn't exist on my server

![image](https://agent-skywalker.github.io/post/ptd/images/snare3.png)

It show's it's vulnerable to remote file inclusion vulnerability


# Exploitation

To exploit this we are going to use `metasploit`  `php_include` module, more info on how to use this module found here [here](https://www.offsec.com/metasploit-unleashed/php-meterpreter/) .

We load the module in metasploit.

![image](https://agent-skywalker.github.io/post/ptd/images/snare4.png)

we can see a great number of options available to us.

![image](https://agent-skywalker.github.io/post/ptd/images/snare5.png)

The most critical option to set in this particular module is the exact path to the vulnerable inclusion point. Where we would normally provide the URL to our PHP shell, we simply need to place the text **XXpathXX** and Metasploit will know to attack this particular point on the site.

![image](https://agent-skywalker.github.io/post/ptd/images/snare6.png)

We run the exploit and gain a meterpreter shell

![image](https://agent-skywalker.github.io/post/ptd/images/snare7.png)

We found FLAG1.txt

![image](https://agent-skywalker.github.io/post/ptd/images/snare8.png)

We stabilize the shell

![image](https://agent-skywalker.github.io/post/ptd/images/snare9.png)

# Privilege Escalation

Looking around in the system we find that the /etc/shadow(File where all password hashes are stored) file is readable and writable. To confirm this i check the file permissions.

![image](https://agent-skywalker.github.io/post/ptd/images/snare10.png)

I cat out the contents of `/etc/shadow` so i can see what hash is being used to hash the passwords

![image](https://agent-skywalker.github.io/post/ptd/images/snare11.png)

I identified the hash as `sha-512`

On my attacker machine i create a password which i will substitute for the root password.

```
mkpasswd -m sha-512 pass
```


![image](https://agent-skywalker.github.io/post/ptd/images/snare12.png)

I copied the contents of `/etc/shadow` file to a new file on my computer named `shadow` then i substituted the generated sha-512 password into root password

![image](https://agent-skywalker.github.io/post/ptd/images/snare13.png)

I started a python http server on my attacker machine so i can download the modified `shadow` file to the target machine.

![image](https://agent-skywalker.github.io/post/ptd/images/snare14.png)

Then i downloaded the modified shadow file into `/etc/shadow`

![image](https://agent-skywalker.github.io/post/ptd/images/snare15.png)

Then we switch user to root user and we got root shell

![image](https://agent-skywalker.github.io/post/ptd/images/snare16.png)





