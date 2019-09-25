# Lab 1: Creating a scalable penetration testing setup 

In this lab we will start creating our penetration testing setup. I'll step through two different setups, one setup using the linux security distro Kali and a second method using the standard Ubuntu linux distro. Students may also want to check out Parrot OS as an alternative to Kali. We'll be running our attack machines as VMs on your main OS.
> You are of course free to create your own custom setup using whatever OS you prefer. 

The Kali/Parrot setup come pre-installed with hundreds of security tools so should avoid you having to install tools each week as we progress. The Ubuntu setup will involve you doing a bit more work each week as you'll need to install every tool we use every week. My personal preference after trying many different setups over the past few years is to use Ubuntu and then just install the tools I actually use and need. Keeping an up to date setup and config file allows me to just duplicate my setup where ever and when ever I need, but this can just as easily be done for Kali/Parrot also, I've just found using Ubuntu to be more stable and scalable long term.
___

## Lab Contents

1. Download and install VMware Workstation 15 Pro
2. Choose and install an OS (Kali or Ubuntu)
    + 2.1 Installing Kali Linux
    + 2.2 Installing Ubuntu Desktop 19.04
    + 2.3 Add network adapters
    + 2.4 Virtual Network Editor
3. Install VMWare Tools
4. Update your OS and take a snapshot
5. Renaming our network interfaces for ease of use
    + 5.1 Adding custom interface names for Kali
    + 5.2 Adding custom interface names for Ubuntu
    + 5.3 Changing IP addresses as needed on the command line
6. Testing your network connections
___

### 1. Download and install VMware Workstation 15 Pro 
We'll be using VMware Workstation Pro (Fusion on Mac) as our main hypervisor to run our VMs. Virtualbox is the free alternative but I've found VMware Workstation to be much beter and allows much better networking and snapshot options. If you really prefer to use Virtiualbox or some other alternative for whatever reasons then there shouldn't be any issues. 
> You should have already received an email link from E-hub or similar to get your copy of VMware Workstation/Fusion Pro. If not let me know and I'll add you.  


> I'll be creating new user accounts for the next couple of weeks only, so don't wait to contact me and then expect me to sort you a licence.

### 2. Choose and install your OS (Kali or Ubuntu) 
In the labs each week I'll be doing any demos using Ubuntu 19.04, Kali VM is probably a better option for the lazy student (or those who just perfer Kali or want to learn the OS) but I won't be troubleshooting an issues people run into while using Kali or other OSs.

#### 2.1 Installing Kali Linux
- Installation guides: https://docs.kali.org/category/installation
- Kali linux downloads page: https://www.kali.org/downloads/
- Kali 2019.3 64bit VMware VM - https://images.offensive-security.com/virtual-images/kali-linux-2019.3-vmware-amd64.7z
- Kali 2019.3 64bit ISO - https://cdimage.kali.org/kali-2019.3/kali-linux-2019.3-amd64.iso
        
#### 2.2 Installing Ubuntu Desktop 19.04
- Installation guide: https://tutorials.ubuntu.com/tutorial/tutorial-install-ubuntu-desktop?_ga=2.250308998.1265951109.1569279578-306165233.1569279578#0
- Ubuntu Desktop 19.04: https://ubuntu.com/download/desktop/thank-you?version=19.04&architecture=amd64
- Ubuntu Desktop 18.04: https://ubuntu.com/download/desktop/thank-you?version=18.04.3&architecture=amd64 

#### 2.3 Add network adapters
Regardless of the OS you've chosen I'm going to recommend you add three network adapters. This setup should allow you connect your attack machine to any type of network (wireless, wired, internal) without reconfiguring much. I'll talked through the differnt network options and the pros and cons and when to use each. 

- Adapter 1 (Wireless Internet): Set to Bridged (goto advanced and record mac the address).
- Adapter 2 (Internal Host only): Set to 'Host-only' (goto advanced and record mac the address).
- Adapter 3 (Wired Ethernet): Set to custom vmnet2 (goto advanced and record mac the address).

> Adapter 1 could just as easily be set to NAT, but becuase of a networking restrictions in the college I'm suggesting you use bridged to avoid any issues.  

> Rememeber to record the which mac is connected to which interface as we'll using this these details later.

#### 2.4 Virtual Network Editor
One of the most common issues I've seen students experience over the years are due to using automatic bridging and then getting connected to a different network interface to what they expected and then carelessly end up scanning the wrong network. We can use VMware's Virtual Network Editor to fine tune our network adapters and what they connect to. The instructions below assume you've created the three adapters I recommended earlier.

- Goto Edit/Virtual network editor on your VMware Workstation.
- Bottom right corner click change settings.
- VMnet0 change from bridged/automatic to bridge to your wireless card and apply changes.
- Add Network VMnet2 and set to bridged with your physical ethernet and apply changes 

> You may not see a physcial ethernet appear if you haven't an ethernet or adapter connected

I'll talk through some of the other options available within the network Editor to better help you allocate Ips and assign adapters.

### 3. Install VMWare Tools
Adding VMware tools (or guest additions on Virtualbox) gives us lots of additional options and much better usability for our VMs, paste and copying betwenn OSs, screen resolutions etc. So it's well worth installing. The latest version of Kali should detect that it is installing as a VM and automatically install some bits to make your life a bit easier.

Boot up your VM and then install VMware Tools (VM/Install VMware Tools).
> Full instructions if needed https://vitux.com/how-to-install-vmware-tools-in-ubuntu/

### 4. Update your OS and take a snapshot
It's always good practice to regularly update and upgrade your system.
> I don't suggest updating/upgrading your sysetm just before an engagement or exam. Always takle a temperary snapshot first just in case something breaks.

```bash
apt-get update && apt-get upgrade -y 
```

Now we have a basic working setup we should take a snapshot of our VM (It's going to get messed up at some point!)
- Right click then Snapshot/Take Snapshot and name as fresh install or similar.

Since this is a clean install you might want to perform some additional tweaks and updates.. maybe even before your snapshot to customise your setup.So maybe check out some of these, and I'm sure there are plenty more.

- Strongly recommend new normal user account for kali users
- Ideas for Ubuntu new install todo list : https://www.techdrivein.com/2019/04/15-things-todo-ubuntu-1904.html
- Ideas for Kali new install todo list: https://www.ceos3c.com/hacking/top-things-after-installing-kali-linux/


### 5. Renaming our network interfaces for ease of use
Like I mentioned before confusion around network adapters and what they connect to causes huge confusion among students, me suggesting three adapters probably doesn't help the issue! We can try make our lives a little bit easier by adding labels to our interfaces. So naming the interface that connects to our wifi network as my_wifi rather than ens3 or eth0. 

#### 5.1 Adding custom interface names for Kali
We can check out the names and details of our interfaces by running the 'ip a' command. To renaming your interfaces on Kali follow the steps shown below.

First create a link file for our wifi interface
```bash
sudo nano /etc/systemd/network/mywifi.link 
```

Next we want to add the following lines, changing the MACAddress and Name to match your our Mac and desired interface name
```bash
[Match]
MACAddress=00:11:22:33:44:55

[Link]
Name=my_wifi
```

Repeat the above process for our two other interfaces. So create two new link files and add the MACAddress and Name. Once you've completed all that you need to run the following command:

```bash
sudo update-initramfs -u 
```
This will embed these updated config files into your initramfs, where they will be applied. Your interfaces should now be renamed, you can check with the 'ip a' command.

#### 5.2 Adding custom interface names for Ubuntu
On Ubuntu we will use the new NetPlan tool to configure our interfaces.

```bash
sudo nano /etc/netplan/01-network-manager-all.yaml 
```

Once you've opened the file you can copy the following into it. Remember to use your own MAC addresses from earlier and you can pick your own interface names just replace the names I've suggested if you wish.

```bash
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
#      gateway4: 10.10.10.1
#      nameservers:
#        addresses: [8.8.8.8, 8.8.4.4]
      match:
        macaddress: 00:0c:29:18:7e:63 # Change Mac to match your own interface
      set-name: my_ethernet
```

> because you are editing/creating a Yaml file make sure you don't use any tabs and use a consistent indentation (I used two spaces for all indents) otherwise you'll get an error

Once you've finished editing the document you just need to apply it using the following command. You'll probably have to restart for the interface names to change, but IP addresses etc should kick in once you've applied the netplan.

```bash
sudo netplan apply
```


#### 5.3 Changing IP addresses as needed on the command line
The following section describes the process of configuring your systems IP address and default gateway needed for communicating on a local area network and the Internet.

For temporary network configurations, you can use the ip command which is also found on most GNU/Linux operating systems. The ip command allows you to configure settings which take effect immediately, however they are not persistent and will be lost after a reboot.

To temporarily configure an IP address, you can use the ip command in the following manner. Modify the IP address and subnet mask to match your network requirements.

```bash
sudo ip addr add 10.102.66.200/24 dev eth0
```

The ip can then be used to set the link up or down.

```bash
ip link set dev eth0 up
ip link set dev eth0 down
```

To verify the IP address configuration of eth0, you can use the ip command in the following manner.

```bash
$ ip address show dev eth0
10: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 00:16:3e:e2:52:42 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.102.66.200/24 brd 10.102.66.255 scope global dynamic eth0
       valid_lft 2857sec preferred_lft 2857sec
    inet6 fe80::216:3eff:fee2:5242/64 scope link
       valid_lft forever preferred_lft forever6
```

To configure a default gateway, you can use the ip command in the following manner. Modify the default gateway address to match your network requirements.

```bash
sudo ip route add default via 10.102.66.1
```

To verify your default gateway configuration, you can use the ip command in the following manner.

```bash
$ ip route show
default via 10.102.66.1 dev eth0 proto dhcp src 10.102.66.200 metric 100
10.102.66.0/24 dev eth0 proto kernel scope link src 10.102.66.200
10.102.66.1 dev eth0 proto dhcp scope link src 10.102.66.200 metric 100 
```

If you no longer need this configuration and wish to purge all IP configuration from an interface, you can use the ip command with the flush option as shown below.

```bash
ip addr flush eth0
```

Further networking configuration bits can be found here: https://help.ubuntu.com/lts/serverguide/network-configuration.html


### 6. Testing your network connections
lets try and use each of our networking adapters to test our connectivity.

Check adapeter 1 can connect to the internet
Check adapter 2 can connect to another VM
Check adapter 3 can connect to our physcial ethernet



