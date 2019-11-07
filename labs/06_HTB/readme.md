# Lab : Getting started with the HacktheBox.eu Website

In this lab we will walk through the process of creating a new account and joining the HTB website. The HTB website allows us access to a number of custom VMs that we can practice our attacks against and perfect our pentesting skills.

If you already have a HTB account you are free to just use that (or create a new one), but read through the rest of the lab to make sure you've completed all the required steps.
___

## Lab Contents

1. Getting a HTB invite code
2. Create a new account
    + 2.1 Read the rules
    + 2.2 Update your profile
3. Download your connection pack
    + 3.1 Connect to the HTB network
    + 3.2 Test connection
4. Check out the list of active machines on HTB
5. Joining the TUDublin dedicated labs
6. What todo on the boxes 
___

### 1. Getting a HTB invite code

To join the hackthebox.eu website and begin attempting any of the challenges and VMs on the site you first need to prove yourself good enough by obtaining an invite code to register and create your new account. 

To get an invite code you need to hack the sign-up page and get yourself an invite code. You can try this out for yourself and try your web hacking for some fun. If you get stuck check out the video on Moodle that should help you get started, but it's much more fun to try it yourself first. 

> See if you can hack the registration page and get yourself an invite!
> https://www.hackthebox.eu/ scroll down to the 'Join Now' section and click 'Join'

### 2. Create a new account
Once you've managed to get yourself an invite code you can join and create a new account. 

> Follow the sign-up steps and create your account, and verify your email. 

#### 2.1 Read the rules
> https://www.hackthebox.eu/home/rules
     
#### 2.2 Update your profile
- Make your account public
- Set the country to Ireland
- Enable 2-factor authentication is you wish
- Upload a profile picture if you wish

> https://www.hackthebox.eu/home/settings

### 3. Download your connection pack

HTB uses a private network that you'll need to connect to using a VPN. (This will be blocked in the college for the next week or so until I get it sorted!) We'll be connecting from our attack VMs (Ubuntu, Kali, Parrot or whatever you're using) 

> So from your attack machine go to the access page and download your connection pack. https://www.hackthebox.eu/home/htb/access

> HTB assumes you have openvpn installed. So you might have to install it to continue.

#### 3.1 Connect to the HTB network

I suggest you create a new folder for your HTB activities and copy your connection pack to that folder. On the command line from within that folder try the following command to try connect your VPN.

```bash 
sudo openvpn your_username.ovpn 
```

#### 3.2 Test Connection

If everything goes well you should be connected, you'll loose connection once you close that terminal window. So open a new window for the remaining commands. If you don;t have these settings you'll need to go back and troubleshoot any error messages you've gotten.

### 4. Check out the list of active machines on HTB

> You can see the currently active HTB machines here: 
> https://www.hackthebox.eu/home/machines

Find the box called 'Wall' and check out it IP (by hovering over it in the list.) When I wrote this it was 10.10.10.157 but it might change.

With your VPN running you should be able to ping this box. your setup is complete and you are connected to HTB.

You now just need to run the 'sudo openvpn your_username.ovpn' command each time you want to reconnect to the HTB network and practice hacking any of the active boxes. Start with your namp commands and away you go.

### 5. Joining the TUDublin dedicated labs

We have our own set of private boxes set up for just our class on the HTB network. To join this class group you need to send me you HTB username. So once you've gotten set up jump over to the class discord chat server and post your HTB username and I'll add you to the class labs.

Once I've added you you should see an option under lab access(https://www.hackthebox.eu/home/htb/access) called 'eu-dedicated-6 Lab Access'. Once you can see this you'll need to click switch.

Once you have switched you'll need to scroll down a bit and download your new connection pack for the class labs. So close your earlier VPN running on the terminal and then reconnect using the new ovpn connection file.

Once you've managed all that you can test your connection by trying to connect to the 'Lame' box which has an IP of 10.10.10.3.. All going well you should now be on the Dedicated TUDublin Labs.

I'll be adding more boxes as we progress.

### 6. What todo on the boxes

Each HTB box has two flags that you need to find. (user.txt and root.txt) You submit these flags on the HTB site to claim a box and earn your points. 

On your main page you should see a link to 'Dedicated Labs > eu-dedicated-6' on this page you can see any boxes that I have active in the class labs. Since these are older boxes the write-ups exist on how to solve them. So try following the walkthroughs available for each box (click on the box link for more details) Just be aware that the official tutorial won't be available just the submitted answers.

