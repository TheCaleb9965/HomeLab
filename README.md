# Introduction
Hello, my name is Caleb. I'm creating this git to follow all changes, updates, issues, and resolutions within my homelab. I'm starting this a little late in the game as far as configuration with my current setup, however I am actively looking for new equipment and software to add/change constantly. Now, as for what's current I will list it below and I will *try* to remember all of the issues I've faced and their resolutions. I will also post what I've learned from each thing. At this time, I'd say I'm around 120-160 hours into designing, and implementing everything here.

# Hardware
Alright lets get right into the thick of it. Currently the network is a standard 3 tier, with an integrated CCNA lab that I will touch on later on.

- Primary Router - Riverbed CX-570
- Primary Switch - Dell N1548P
- CCNA Lab Equpment: 4x Cisco 1841, 1x Cisco 24 Port 3750, 1x Cisco 24 Port 3560, 2x Cisco 48 Port 2950 (Might be overkill but I got all that for 80 dollars so I'll find a use for it)
- ISP gateway - Provided by Cox
 
## Core
For my primary firewall/router I have a riverbed CX-570 running OPNsense. I originally had another firewall I had received for free, but there was an issue and I was not able to get so much as a bios boot no matter the console/BAUD settings. Because of this, I went on the hunt. My original plan was to find some sort of cisco ISR (due to me working towards my CCNA and having prior experience) and then a palo alto 820 (Also prior experience and PCNSE is up on the list for studying after CCNA). I was looking at somewhere in the range of 250 dollars or so for those two when I saw this device come up on facebook marketplace for 100 (naturally I got him down to 80 bucks and picked up that day).

And thus begins the woes of learning a new platform and putting my skills to the test. I'm going to touch each individual issue later but that should give you an idea of how this went. 

## Distribution

For my switch I have a Dell N1548P that I received from a client previously, this device actually has propelled my career in networking. As I was originally studying for my CCNA it forced me to learn the concepts as opposed to just what Cisco wants since it works *relatively* the same but just enough different to be ... quite an annoyance.
I believe here is a good place to add my VLAN assignments (note these may change depending on my mood.
- VLAN 10 - 10.0.10.0/24 - Primary Management
- VLAN 20 - 10.0.20.0/24 - Primary Network Services
- VLAN 30 - 10.0.30.0/24 - VMs
- VLAN 40 - 10.0.40.0/24 - Lab MGMT
- VLAN 50 - 10.0.50.0/24 - Users

I realize that at my current size 5 /24 networks is somewhere in the range of 1250 useable IPs and theres no possible way I could reach that but I clearly have plenty of room to waste some IPs :P

## VMs

So currently I have 2 Proxmox Nodes (i.e. old laptops) in my lab, specs are as follows;

### HMR-SERV-1 - Intel i7-9750H (12 core), RTX 2060, 16gb RAM, 500gb Hard Drive
- Monitoring
- Twingate Connector
- Ticketing Software (How I keep track of everything I'm doing, forces me to have some self discipline)
- Private Rocket Chat
- Minecraft Server (doesn't really get used, just have the server setup and space reserved for when I feel like playing)

### HMR-SERV-2 - Intel i3-7100u (4 core), minimal graphics, 6gb RAM, 1tb Hard Drive
- Windows Server 2019 (DNS, going to be setting up DHCP within here also in the future

### HMR-SERV-3 (To be added) - Intel i3-1215u (6 core), minimal graphics, 8gb RAM, 250gb Hard Drive
No current VMs, infact it is still running windows but lets list the plans:
- Migrate Windows Server 2019 VM to this device
- Upon doing so we're going to remove 4gb RAM and the hard drive from HMR-SERV-2, reformat the drive and use a usb-c to sata adapter, and then add them to this device. (RAM is compatible) at that point this device will become HMR-SERV-2 and that device will become 3
- That device will be left to exclusively run a secondary Twingate Connector and maybe a debian host I can use to allow friends and such to practice their CCNA skills.

# Network Design
Here is the current network diagram, I'm still adjusting service IPs and as I do this will be adjusted.


<img
  alt="home lab"
  src="https://github.com/user-attachments/assets/37edd5c9-2f95-4b92-993f-15a5eb7b631a"
  style="max-width: 1000px; width: 100%; height: auto;"
>

# Ongoing Work
## 2/9/26
Well wrapping up today around 11pm after a VERY long day working on the lab. Last night I was working on setting up AD for LDAP and RADIUS authentication, well when I tested a user on the ldap within my OPNSense I suddenly didn't have access to the router at ALL. This includes root access within the CLI which is supposed to be unchanged regardless of GUI access/authentication settings. Attempted a password recovery using an image on a USB key which failed, thus giving me no choice but to reset everything to a fresh image.

I believe that this was caused by some glitch with the replication from AD to the OPNSense users. However, not having access I wasn't able to dig around and find out.

At this point I decided that I would go ahead do a couple of housekeeping tasks, such as actually setting up my subinterfaces as opposed to using a cable for each VLAN, which mostly frees up some cable space in the rack, but also gives me back some of my port density on my OPNSense router if I wanted to create any sort of DMZs or separate LANs in the future. 

Lets go ahead and list everything I actually accomplished today
- OPNsense base configuration
- Created subinterfaces for VLANs 10/20/30/40 on LAN port
- DHCP scopes for VLAN 10 and 20
- Made HMR-SERV-2 VLAN aware, setting proxmox host IP to the proper 10.0.10.251 and giving the adapter for VLAN 20 the IP 10.0.20.2 (unsure if this is needed, will research at a later time.)
- Adjusted hostname and FQDN within proxmox on HMR-SERV-2, HMR-SERV-1 is still pending.
- Adjusted IP for DNS server to 10.0.20.3 to be inline with VLAN scheme.
- Issue (resolved) - DHCP service in OPNSense was not allowing subinterfaces to run the DORA process, thus not assigning IPs. I had it set to the original LAN as opposed to allowing each of the subinterfaces on there instead. This has now been resolved.
- Housekeeping - connected all ports from OPNSense to switch port and generated traffic. used mac table to then determine which port is which NIC slot on the router (Left to Right is IGB4,IGB5,IGB0,IGB1,IGB2,IGB3) and physically labelled accordingly.
- Housekeeping - upon configuring subinterfaces, cleaned up cables out of rack, hopefully by the end of the week I'll be in a place where I'm happy to take a picture of this thing.
- Brought diagram mostly up to date, as I re onboard different services I will add them in there but everything that is currently online is there.
- Oh yeah and most importantly created ALL the firewall rules again, needless to say I will be downloading some sort of backup on my PC before any changes going forward...guess thats one of those things you have to get burned to learn.
And that's about it, I'm sure I'm missing something considering the sheer amount of work I did today but I'd say thats the important bit.
