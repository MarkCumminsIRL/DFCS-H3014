# Solving : De-ice 1.100 Vulnerable VM


## Lab Overview
In this lab we are going to put all of the bits we've covered to date together and perform a mini Pentest against the vulnerable De-ice 1.100 VM.

As part of the walk through we'll also introduce the password brute forcing tools Hydra and Medusa. This Walk through will be a good starting point and template for any further Pentests you need to perform.


## Downloading the iso 
Step 0. To perform this lab we need to create a new VM and install the de-ice 1.100 image, there are several versions so make sure to use the 1.100 version.

> Image can be downloaded from here: http://hackingdojo.com/downloads/iso/De-ICE_S1.100.iso
> I've also added a version to Moodle Labs section which might be quicker to download 

Step 1. Once downloaded you'll need to create a new 'Other Linux 3.x kernal' VM and set the network adapter to host only. 


## SCENARIO
The scenario for this LiveCD is that a CEO of a small company has been pressured by the Board of Directors to have a penetration test done within the company. The CEO, believing his company is secure, feels this is a huge waste of money, especially since he already has a company scan their network for vulnerabilities (using nessus). To make the BoD happy, he decides to hire you for a 5-day job; and because he really doesn't believe the company is insecure, he has contracted you to look at only one server - an old system that only has a web-based list of the company's contact information.

The CEO expects you to prove that the admins of the box follow all proper accepted security practices, and that you will not be able to obtain access to the box. Prove to him that a full penetration test of their entire corporation would be the best way to ensure his company is actually following best security practices.



## CONFIGURATION
This LiveCD is pre-configured with an IP address of 192.168.1.100

We will need to ensure that both our Attack machine and the De-Ice image are connected to the host only network adapter.
The easiest solution is to disable all of your adapters except the host only one, this will make troubleshooting much easier.

Step 2. -- Set both your Attack machine and De-Ice images to only use host only network adapters.

We also need to ensure that our Attack machine gets an IP address from the same range 192.168.1.xxx. We can do this a couple of different ways.
First we can set the range of the DHCP server in our Hypervisor to the matching range, then start our Attack VM up and it should receive the required IP.
Secondly we can set the IP address on our Attack VM manually, to again match our required range.

Step 3. -- Set the DHCP range in your Hypervisor to use 192.168.1.0 network

Step 4. -- Start up both of your VMs
> Once installed you should see a 'slax login:' prompt



## TESTING OUR SETUP
To check our setup we'll use our ARP scanning tool netdiscover (sudo apt-get install netdiscover if you don't have it installed already)

Step 5. -- Test your setup is working by using netdiscover to see connected devices. netdiscover -r 192.168.1.0/24 (you may need to use -i interface_name )

Step 6. -- If you don't see the 192.168.1.100 machine listed, then you'll need to troubleshoot steps 2-4 before repeating step 5 and continuing.



## Stage 1 - Port Scanning
Normally we'd just try and ping our target machine to check if we have connectivity. So let's try it now.

Step 7. -- Ping 192.168.1.100 from our attack machine.

You should notice that in this case the ping doesn't work (This is best practice, to disable ICMP echo requests).
This is why we should always use more than one tool when checking. As we're directly connected to the same network we can be sure that our netdiscover command (which uses ARPs) will always work. We could also have used an nmap -sP 192.168.1.100 (to check if the host is active)

OK our next step is check which services (open ports) are running on the target machine. Nmap should be our go to tool here.

Step 8. -- nmap -sV -v -n 192.168.1.100   (-sV service detection for detailed port info, -v verbose output, -n disable name service lookup)

Step 9. -- Create a list of all open services that we can use later to check for vulnerabilities.

At this stage we should have a list of open ports, of course these are just the open ports from the 1000 most common ports that we scanned.
To be sure to find all open ports we would normally run a full port scan and UDP scan also. These take time, so we can investigate the open ports we have found so far while these scans run in the background.

Step 10. -- Open up a separate terminal windows and perform a full port scan : nmap -sV -p- -n 192.168.1.100 (-p- scan all ports)

Step 11. -- Again open a another separate terminal window and perform a full UDP scan: nmap -sUV -p- -n 192.168.1.100 (-sUV scan UDP ports..this is very slow)

Step 12. -- Once our scans complete check if we have found any additional open ports and add them to the list created in Step 9.


---------------------------------------------------------------------------------------------------------
Stage 2 - Check found Services for vulnerabilities
---------------------------------------------------------------------------------------------------------

Once we've found a list of open ports we would normally check each service version to see if there are any known vulnerabilities that we might be able to use to simply exploit the system.

In this case we've been told that the network is regularly scanned with Nessus (our go to vulnerability scanner) so it's a safe bet that we won't (or shouldn't) find any vulnerabilities.

So we'll leave this for now.. we might come back to look at it again some other time.


---------------------------------------------------------------------------------------------------------
Stage 3 - Recon (part 1)
---------------------------------------------------------------------------------------------------------

While recon is usually the first step of any pen test, the process is circular, as we find more details it gives us more areas to go back and recon. in this case the list of open services.

You should have noticed that both http port 80 and https port 443 are open of the target machine. So lets check out what webpages are there and try get some more info to help with our attack. (http and https can often be different sites)

Step 13. -- Open your browser and visit both https://192.168.1.100 and https://192.168.1.100

You should be able to connect on port 80 using normal http but you'll notice that https on port 443 doesn't respond. It's worth having a closer look at the namp outputs for both ports. You'll notice that port 443 is actually listed as closed. In this case it was likely open at some stage and the service behind it shutdown or crashed, you'll notice several of the ports are listed as closed. You wont be able to connect to a closed port, so look at the nmap output carefully.

Step 14. -- Mark the services on our list from step 9 which are listed as closed so we know not to try these.

Step 15. -- Try connecting to the ftp service to see if it allows anonymous access.

At this stage you should have the following ports left in your list as possible ways into this system
22/tcp  open   ssh      OpenSSH 4.3 (protocol 1.99)
25/tcp  open   smtp     Sendmail 8.13.7/8.13.7
80/tcp  open   http     Apache httpd 2.0.55 ((Unix) PHP/5.1.2)
110/tcp open   pop3     Openwall popa3d
143/tcp open   imap     UW imapd 2004.357

22/SSH is obviously a good choice to try and use to gain access with, but we'll need to find some user names or passwords to help us.
25/smtp Seems active also so maybe we can also use this to help us.. maybe use this to help us enumerate usernames.
80/Web seems to have a basic webpage that is worth looking through for more details that might help us.
110/143/Pop/imap Both are mail protocols that also allow us to authenticate and maybe login, so these are other options for trying to connect to.

---------------------------------------------------------------------------------------------------------
Stage 4 - Recon (part 2)
---------------------------------------------------------------------------------------------------------

Lets go look through the Website and see what we can find. We wont use any fancy tools just basic observation for now.

Step 16. -- Check all pages on the website, looking for anything that might assist us. (don't look at the hints page)

We should be able to find a webpage listing some employees, which is a good place to start, and we can use this list to create our username list.

Step 17. -- List all of the employees found on the webpages in a single text file in the format firstname surname

You should end up with a list similar to the following
charlie o
marie mary
pat patrick
terry thompson
ben benedict
erin gennieg
paul michael
ester long
adam adams
bob banter
chad coffee

It might be worth arranging the list in order you think most useful, so admins etc to the top, as the best accounts to break if we can. So Adam Adams is a good bet, its also worth noting that Bob Banter is listed as an intern, a likely source of possible weakness...

Step 18. -- Create your own username list based on all the combinations of employee names you can think of.

I tend to use a popular script for this, that creates username lists for you from a input file of firstnames surnames.
I'll make the script available on Moodle.


---------------------------------------------------------------------------------------------------------
Stage 5 - ENUMERATION
---------------------------------------------------------------------------------------------------------

Ok so at this stage we should have a list of possible usernames, Ideally we want to reduce this list as much as possible before we attempt using the list for any brute forcing attempts.

Step 19. -- Attempt to use smtp-user-enum to enumerate any of your usernames, and hence reduce your username list


---------------------------------------------------------------------------------------------------------
Stage 6 - BRUTE FORCE
---------------------------------------------------------------------------------------------------------

OK so it looks like we've exhausted all our other options, time to use what we've found so far to help us attempt a brute force login.
We could try this against the SSH, POP or IMAP services but SSH seems like the obvious choice.

Step 20. -- Try to brute force the connection using your favourite tool (hydra etc.)

You might have to reduce the number of threads your using as it might over whelm the target. I've given the command I use below.
hydra -e sr 192.168.1.100 ssh -L usernames.txt -F -V -t 6 -u
hydra -P /usr/share/wordlists/rockyou.txt 192.168.1.100 ssh -L usernames.txt -F -V -t 6 -u

Step 21. -- Check out and understand what each of the options I've used mean

Once we've found the first user (-F to exit when user found) we should be able to figure out the username format and we could edit our username list. But it's worth using our new SSH login to see what we can find on the system.


---------------------------------------------------------------------------------------------------------
Stage 7 - SSH ONTO TARGET
---------------------------------------------------------------------------------------------------------

At this stage we should have found the weak password for the user bbanter. This reveals to us the username format. but for now lets SSH onto the target as bbanter and see what we can find on the system.

Step 22. -- SSH on the Target system: ssh 192.168.1.100 -l bbanter

Once we're in we can look around and see what if anything we can find. Ideally we'd like to get a copy of the shadow file, but since we're not root we wont be able to do that. But we can at least look at the /etc/passwd file and get an accurate list of users, and also the /etc/group file to see what group each user is a member of.

Step 23. -- Look at both the /etc/passwd file and /etc/group file

Step 24. -- What group is each user a member of?

We should see that to get access to the shadow file and hence the root password we're going to need to use the aadams account as he is a member of the wheel account which has root permissions.

---------------------------------------------------------------------------------------------------------
Stage 8 - BRUTE FORCE AADAMS ACCOUNT
---------------------------------------------------------------------------------------------------------

We're back to brute forcing the SSH login but this time at least we a specific user account to target.

Step 25. -- Brute force the aadams SSH account
hydra -P /usr/share/wordlists/rockyou.txt 192.168.1.100 ssh -l aadams -F -V -t 6 -u

OK this is going to take a while, but if the password is in our list it will hopefully pop out with the correct password.
So after some time we should hopefully see the aadams password of nostradamus

We can now login using the found username and password and try get the shadow file.

Step 26. -- SSH onto our target machine using the aadams credentials
ssh 192.168.1.100 -l aadams

Step 27. -- Get a copy of the /etc/shadow file

---------------------------------------------------------------------------------------------------------
Stage 9 - GET ROOT PASSWORD
---------------------------------------------------------------------------------------------------------

We finally have a copy of the shadow file, and we can see the hashed password for the root account.
Our next step is to try break this hash and get the password.

We can just copy the line from the shadow file for the accounts we want to brute force and save them a text file. We can then use john the ripper to brute force the hashes for us.

Step 28. -- Use John to break the shadowed passwords.
john shadow.txt (this will use john's own built in word list)
john --wordlist=/usr/share/wordlists/rockyou.txt (to use the rockyou word list)

Again this step will take a while but it should finally pop out with the root password.  
