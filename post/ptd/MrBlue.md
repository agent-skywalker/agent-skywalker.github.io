![image](https://agent-skywalker.github.io/post/ptd/images/mrblue1.png)
nmap scan for open ports
```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-05-10 01:27 WAT
Initiating Ping Scan at 01:27
Scanning 10.150.150.242 [2 ports]
Completed Ping Scan at 01:27, 0.30s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 01:27
Completed Parallel DNS resolution of 1 host. at 01:27, 0.31s elapsed
Initiating Connect Scan at 01:27
Scanning 10.150.150.242 [1000 ports]
Discovered open port 80/tcp on 10.150.150.242
Discovered open port 445/tcp on 10.150.150.242
Discovered open port 53/tcp on 10.150.150.242
Discovered open port 3389/tcp on 10.150.150.242
Discovered open port 139/tcp on 10.150.150.242
Discovered open port 135/tcp on 10.150.150.242
Discovered open port 49155/tcp on 10.150.150.242
Discovered open port 49152/tcp on 10.150.150.242
Discovered open port 49158/tcp on 10.150.150.242
Discovered open port 1433/tcp on 10.150.150.242
Completed Connect Scan at 01:28, 21.20s elapsed (1000 total ports)
Nmap scan report for 10.150.150.242
Host is up (0.31s latency).
Not shown: 656 closed tcp ports (conn-refused), 334 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE
53/tcp    open  domain
80/tcp    open  http
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
1433/tcp  open  ms-sql-s
3389/tcp  open  ms-wbt-server
49152/tcp open  unknown
49155/tcp open  unknown
49158/tcp open  unknown

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 21.86 seconds

```

Full nmap scan

```
PORT      STATE SERVICE            VERSION
53/tcp    open  domain             Microsoft DNS 6.1.7601 (1DB1446A) (Windows Server 2008 R2 SP1)
| dns-nsid: 
|_  bind.version: Microsoft DNS 6.1.7601 (1DB1446A)
80/tcp    open  http               Microsoft IIS httpd 7.5
|_http-server-header: Microsoft-IIS/7.5
| http-methods: 
|_  Supported Methods: HEAD OPTIONS
|_http-title: Site doesn't have a title (text/html).
135/tcp   open  msrpc              Microsoft Windows RPC
139/tcp   open  netbios-ssn        Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds       Windows Server 2008 R2 Enterprise 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
1433/tcp  open  ms-sql-s           Microsoft SQL Server 2012 11.00.2100.00; RTM
| ms-sql-info: 
|   10.150.150.242:1433: 
|     Version: 
|       name: Microsoft SQL Server 2012 RTM
|       number: 11.00.2100.00
|       Product: Microsoft SQL Server 2012
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
|_ssl-date: 2024-05-10T01:02:17+00:00; +29m36s from scanner time.
| ms-sql-ntlm-info: 
|   10.150.150.242:1433: 
|     Target_Name: MRBLUE
|     NetBIOS_Domain_Name: MRBLUE
|     NetBIOS_Computer_Name: MRBLUE
|     DNS_Domain_Name: MrBlue
|     DNS_Computer_Name: MrBlue
|_    Product_Version: 6.1.7601
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Issuer: commonName=SSL_Self_Signed_Fallback
| Public Key type: rsa
| Public Key bits: 1024
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2020-03-25T14:11:19
| Not valid after:  2050-03-25T14:11:19
| MD5:   6168:8dc8:79c8:5e9a:9620:6f0f:de6a:f664
|_SHA-1: 39eb:98b3:0b8c:2ffd:dba3:5509:71d5:489a:d5e1:7f2b
3389/tcp  open  ssl/ms-wbt-server?
49152/tcp open  msrpc              Microsoft Windows RPC
49155/tcp open  msrpc              Microsoft Windows RPC
49158/tcp open  msrpc              Microsoft Windows RPC
Service Info: Host: MRBLUE; OS: Windows; CPE: cpe:/o:microsoft:windows_server_2008:r2:sp1, cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2024-05-10T01:02:07
|_  start_date: 2020-03-25T14:11:23
| smb2-security-mode: 
|   2:1:0: 
|_    Message signing enabled but not required
| smb-os-discovery: 
|   OS: Windows Server 2008 R2 Enterprise 7601 Service Pack 1 (Windows Server 2008 R2 Enterprise 6.1)
|   OS CPE: cpe:/o:microsoft:windows_server_2008::sp1
|   Computer name: MrBlue
|   NetBIOS computer name: MRBLUE\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2024-05-10T01:02:07+00:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_clock-skew: mean: 29m36s, deviation: 3s, median: 29m35s

```

So there's a web server at port 80

Navigating to the web server is this page

![image](https://agent-skywalker.github.io/post/ptd/images/mrblue2.png)

Directory brute forcing didn't yield any results
Nothing in the robots.txt file

Reading the page source i found a hint

![image](https://agent-skywalker.github.io/post/ptd/images/mrblue3.png)

The famous eternal blue. So the machine is likely vulnerable to eternal blue 

So i ran metasploit and searched for `ms17_010_eternalblue`

and set the `rhost` to be the target and `lhost` to be my tun0 address and ran the exploit and i got a shell

![image](https://agent-skywalker.github.io/post/ptd/images/mrblue4.png)


Listing the contents of the Desktop folder of user `Administrator.GNBUSCA-W054` i got `FLAG34.txt`

![image](https://agent-skywalker.github.io/post/ptd/images/mrblue5.png)


