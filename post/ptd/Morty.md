![image](https://agent-skywalker.github.io/post/ptd/images/morty1.png)
# Reconnaissance 
Full Nmap scan

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 e8:60:09:66:aa:1f:e8:76:d8:84:16:18:1c:e4:ee:32 (RSA)
|   256 92:09:d3:0e:f9:47:48:03:9f:32:9f:0f:17:87:c2:a4 (ECDSA)
|_  256 1d:d1:b3:2b:24:dc:c2:8a:d7:ca:44:39:24:c3:af:3d (ED25519)
53/tcp open  domain  ISC BIND 9.16.1 (Ubuntu Linux)
| dns-nsid: 
|_  bind.version: 9.16.1-Ubuntu
80/tcp open  http    Apache httpd 2.4.41
| http-ls: Volume /
| SIZE  TIME              FILENAME
| 147   2020-06-10 11:25  note.html
|_
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Index of /
| http-methods: 
|_  Supported Methods: OPTIONS HEAD GET POST
Service Info: Host: 127.0.1.1; OS: Linux; CPE: cpe:/o:linux:linux_kernel

NSE: Script Post-scanning.
Initiating NSE at 19:02
Completed NSE at 19:02, 0.00s elapsed
Initiating NSE at 19:02
Completed NSE at 19:02, 0.00s elapsed
Initiating NSE at 19:02
Completed NSE at 19:02, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 31.29 seconds
                                                                  
```

From the nmap scan it show's there's a web server running on port 80. Navigating to port 80 i found a note.

![image](https://agent-skywalker.github.io/post/ptd/images/morty2.png)

Opening the note i found this inside

![image](https://agent-skywalker.github.io/post/ptd/images/morty3.png)

There's a hint saying they have configured the domain on `mortysserver.com`  adding it to my `/etc/hosts` file with the IP of the box i was able to navigate to the domain.

![image](https://agent-skywalker.github.io/post/ptd/images/morty4.png)

On the webpage it gave a hint that `Fl4sk#!` might be a password so since ssh was opened i tried to ssh with user `morty` or  `rick` but it didn't work.

So i used 

```
wget http://mortysserver.com/screen.jpeg
```

To download the image on home screen on to see if i can get information from there since i was getting anything.

After downloading the image i used exiftool but didn't get anything so i used steghide

```
steghide extract -sf screen.jpeg
```

it prompted me to enter passphrase so i used the `Fl4sk#!` and a text file was hidden in the image

![image](https://agent-skywalker.github.io/post/ptd/images/morty5.png)

Catting out the contents of the extracted file was a username and password

![image](https://agent-skywalker.github.io/post/ptd/images/morty6.png)

So i tried to use the credentials to log into ssh but it didn't work

Since going back to the nmap scan results port 53 was open which is dns. To enumerate this i used dig in a method called zone transfer (Zone transfer is **the process of copying the contents of the zone file on a primary DNS server to a secondary DNS server**.)

```
dig axfr @10.150.150.57 mortysserver.com
```

![image](https://agent-skywalker.github.io/post/ptd/images/morty7.png)


We discover a new subdomain 

```
rickscontrolpanel.mortysserver.com
```

# Exploitation 

Adding the new domain to my host file i was able to navigate to it

![image](https://agent-skywalker.github.io/post/ptd/images/morty8.png)

It is a login page and it has Flag 1 on it

Using the credentials gotten from steghide i was able to login

![image](https://agent-skywalker.github.io/post/ptd/images/morty9.png)

And Flag2 was there

Looking around i was able to identified the version of phpmyadmin 

![image](https://agent-skywalker.github.io/post/ptd/images/morty10.png)

Looking for exploits for the phpmyadmin version i found one [here](https://www.exploit-db.com/exploits/50457)

Running the exploit i was able to execute a command on the system

![image](https://agent-skywalker.github.io/post/ptd/images/morty11.png)

Now to get a shell 

I tried using netcat to get a reverse shell but it didn't work

so i used a php reverse to do this i had to list the current directory using the exploit 

![image](https://agent-skywalker.github.io/post/ptd/images/morty12.png)

it shows i can upload a php web reverse shell and access it 

so i set up a python web server on my machine and using the RCE exploit to download the php reverse shell payload to the target machine

![image](https://agent-skywalker.github.io/post/ptd/images/morty13.png)

I then set up my netcat listener and navigated to `http://rickscontrolpanel.mortysserver.com/php-reverse-shell.php`  and i got a shell

![image](https://agent-skywalker.github.io/post/ptd/images/morty14.png)

I stabilize then navigated to the user in the home directory and found the last flag FLAG31 

![image](https://agent-skywalker.github.io/post/ptd/images/morty15.png)

No Privilege Escalation here
