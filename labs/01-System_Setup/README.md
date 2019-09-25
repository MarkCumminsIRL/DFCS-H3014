# Lab 1: Creating a scalable penetration testing setup 

In this lab we will start creating our penetration testing setup. I'll step through two different setups, one setup using the linux security distro Kali and a second method using the standard Ubuntu linux distro. Students may also want to check out Parrot OS as an alternative to Kali. We'll be running our attack machines as VMs on your main OS.
> You are of course free to create your own custom setup using whatever OS you prefer. 

The Kali/Parrot setup come pre-installed with hundreds of security tools so should avoid you having to install tools each week as we progress. The Ubuntu setup will involve you doing a bit more work each each week as you'll need to install every tool we use every week. My personal preference after trying many different setups over the years is to use Ubuntu and then just install the tools I actually use and need. Keeping an up to date setup and config file allows me to just duplicate my setup where ever and when ever I need, but this can just as easily be done for Kali/Parrot also, I've just found using the Ubuntu to be more stable and scalable long term.
___

## Lab Contents

1. Download and install VMware Workstation 15 Pro. 
___

### 1. Download and install VMware Workstation 15 Pro. 
We'll be using VMware Workstation Pro (Fusion on Mac) as our main hypervisor to run our VMs. Virtualbox is the free alternative but I've found VMware Workstation to be much beter and allows much better networking and snapshotting options. If you really prefer to using Virtiualbox for whatever reasons then there shouldn't be any issues. 
> You should have already received an email link from E-hub or similar to get your copy of VMware Workstation/Fusion Pro. If not let me know and I'll add you.
> I'll be creating new user accounts for the next couple of weeks only, so don't wait to contact me and then expect me to sort you a licence.



2. Choose an OS (Kali or Ubuntu) (I'M going to step through the labs using Ubuntu 19.04, Kali VM is probably a better option for the lazy student)
    
    2.1 Pick Kali Image to install
        - Kali linux downloads page: https://www.kali.org/downloads/
        - Kali 2019.3 64bit VMware VM - https://images.offensive-security.com/virtual-images/kali-linux-2019.3-vmware-amd64.7z
        - Kali 2019.3 64bit ISO - https://cdimage.kali.org/kali-2019.3/kali-linux-2019.3-amd64.iso
        - Installation guides: https://docs.kali.org/category/installation

    2.2 Pick a Ubuntu Image to install
        - Ubuntu Desktop 18.04: https://ubuntu.com/download/desktop/thank-you?version=18.04.3&architecture=amd64 
        - Ubuntu Desktop 19.04: https://ubuntu.com/download/desktop/thank-you?version=19.04&architecture=amd64
        - Installation guide: https://tutorials.ubuntu.com/tutorial/tutorial-install-ubuntu-desktop?_ga=2.250308998.1265951109.1569279578-306165233.1569279578#0

    2.3 Add network adapters
        - Regardless of the OS you've chosen I'm going to recommend you add three network adapters.
        - Adapter 1 (Wireless Internet): Set to Bridged (goto advanced and record mac the address).
        - Adapter 2 (Internal Host only): Set to 'Host-only' (goto advanced and record mac the address).
        - Adapter 3 (Wired Ethernet): Set to custom vmnet2 (goto advanced and record mac the address).
    
    2.4 Virtual Network Editor
        - Goto Edit/Virtual network editor on your VMware Workstation.
        - Bottom right corner click change settings.
        - VMnet0 changed from bridged/automatic to bridge to your wireless card and apply changes.
        - Add Network VMnet2 and set to bridged with your ethernet and apply changes (may not appear if you haven't an ethernet or adapter connected).



3. Install VMWare Tools
    
    3.1 Boot up your VM and then install VMware Tools (VM/Install VMware Tools).
        - Full instructions if needed https://vitux.com/how-to-install-vmware-tools-in-ubuntu/



4. Sign up for a Git-hub account

    4.1 If you don't already have a github account then go and register for a new account
        - https://education.github.com/students (Get Benefits)



5. Running a remote setup script from Github

    5.1 Regardless of the OS you've chosen I suggest you start to develop a custom setup script for yourself
        - You should update your script every time you make any command line changes to your OS.
        - This will allow you to just recreate a new VM at any stage and get it back to your own custom setup.
        - A setup script in just a list of command line instructions you want to execute.
    
    5.2 Try execute a remote Github script
        - Type the following command: 
            - wget -qO- https://raw.githubusercontent.com/MarkCumminsIRL/DFCS-H3014/master/basicsetup.sh | bash
            - Using wget or curl to directly fetch and execute a script can be dangerous so always check the script carefully or only ever use your own scripts.

    5.3 Write your own basic setup script and try execute it
        - Create a new Gist on github for your setup.sh file 
        - You'll need a direct link to the raw file

    5.4 Update your script
        - Each week as you progress you should keep updating and customising your setup script.



6. Update your OS and take a snapshot

    6.1 Always good to regularly update and upgrade your system.
        - apt-get update && apt-get upgrade -y (add it to your setup script)
        

    6.2 Now we have a basic working setup we should take a snapshot of our VM (It's going to get messed up at some point!)
        - Right click Snapshot/Take Snapshot and name as fresh install or similar.

    6.3 Perform any additional setup tweaks you want to customise your setup
        - Strongly recommend new normal user account for kali users
        - Ideas for Ubuntu new install todo list : https://www.techdrivein.com/2019/04/15-things-todo-ubuntu-1904.html
        - Ideas for Kali new install todo list: https://www.ceos3c.com/hacking/top-things-after-installing-kali-linux/
        - Remember to add them to your setup script.


7. Renaming our network interfaces for ease of use.

    7.1 The single biggest issue faced by students is confusion about their network interfaces. 
        - We can try make it a bit easier for ourselves by adding labels to our interfaces.
    
    7.2 Adding custom interface names for Kali

    7.3 Adding custom interface names for Ubuntu

    7.4 Changing IP addresses as needed on the command line





ip -c address
sudo netplan apply
sudo gedit /etc/netplan/01-network-manager-all.yaml 

# Let NetworkManager manage all devices on this system
network:
  version: 2
  renderer: networkd 

#   Using DHCP to get an IP address from our bridged Wireless Connection
  ethernets:
    my_wifi:                          # Can set your own interface names
      dhcp4: true
      match:
        macaddress: 00:0c:29:18:7e:4f # Change Mac to match your own interface
      set-name: my_wifi               # Can set your own interface names

#   Using DHCP to get an IP address on our NATed internal host network 
    my_vms:
      dhcp4: true
      match:
        macaddress: 00:0c:29:18:7e:59 # Change Mac to match your own interface
      set-name: my_vms

#   Setting a static IP and network settings for our wired Ethernet Connection
    my_ethernet:
#     dhcp4: true
      addresses:
        - 10.10.10.2/24
      gateway4: 10.10.10.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
      match:
        macaddress: 00:0c:29:18:7e:63 # Change Mac to match your own interface
      set-name: my_ethernet


https://help.ubuntu.com/lts/serverguide/network-configuration.html


cat /etc/systemd/network/10-uplink0.link 
[Match]
MACAddress=00:0d:b9:49:8a:18

[Link]
Name=uplink0
Don’t forget to run update-initramfs -u afterwards to embed these updated config files into your initramfs, where they will be applied.



setup networking.. and test it...
connect two laptops together...
install another vm
connect to internet

how vmware networking works.



wget -qO- https://gist.githubusercontent.com/MarkCumminsIRL/5fd0d03e5252a9b80e3986027fe8dd37/raw/a7e23e1bde9421ab3f1dabf3c21cd637081cf21d/setup.sh|bash



7. Create setup script

8. Create dotfiles

9. Sign-up to Hackthebox.eu
    
    6.1 Send me your user details
11. Look at recon tools.
