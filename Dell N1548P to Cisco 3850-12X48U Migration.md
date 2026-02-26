This migration is scheduled for 2/26, and will have an impact on the entire network.

Note: This MOP is for my personal Home Lab. There might be some humor that wouldn't be included on a proper MOP

During this time we will also be making some cabling and configuration changes to bring the rack more in line with how I envisioned it, as opposed to the state it is in now.

# Switch Verification

Now before we even attempt to install this we're going to boot this switch up and confirm it has an IP base license (I purchased that specifically for future projects)

We will also go ahead and confirm the IOS version and compare to the configs listed below, just to make sure there isn't any differences in commands (This will be noted in an update at the end of the document)

Beyond this we can just move forward with installation

# Rack Changes and Port Changes:

1. Physical Layer Tasks
	1. Remove dell N1548P
	2. Install Cisco 3850-12X48U into rack
		1. will require power down and removal of Riverbed Router due to having incorrect mounting equipment, will try to resolve day of.
	3. Run 4x cat6 between patch panel 1 (Port 1-4) & patch panel 2 (Port 1-4)
		1. This will be our layer 2 connection between the 4 routers, allowing us to have some flexibility in the CCNA Lab Environment.
	4. Run 3x Cat6 between Patch Panel 1 (Port 21-23) and Patch Panel 2 (Port 5-7)
		1. This will serve as our connections between HMR-SERV-1,2,3
	5. Insert rj45 female-female keystone in Patch Panel 1 Port 24
		1. This will give us an easy jumper from our Cox Gateway for now, as the whole rack will be moving in the near future
	6. Run 4x Cat6 between Patch Panel 1 (Port 5-8) to patch panel 2 (Port 5-8)
		1. This will serve as our connections from our CCNA Lab Routers to our CCNA Lab Switches.
	7. Reconnect patch panel port 1-4 to switch port 1-4 on new switch
	8. Connect patch panel port 21-23 to switch port 45-47
	9. Connect patch panel port 24 to WAN on riverbed
	10. Connect LAN port on riverbed to Port 48 on switch

# Now for the good stuff, since this MOP is a one time change, I will just paste current configs below...
These are our current configs so we will be 

```
vlan 10,20,30,40,500
exit
vlan 10
name "primary-management"
exit
vlan 20
name "primary-network-services"
exit
vlan 30
name "VMs"
exit
vlan 40
name "lab-management"
exit
vlan 500
name "test"
exit
ip telnet server disable
hostname "HMR-CSW-1"
slot 1/0 4    ! Dell EMC Networking N1548P
sntp unicast client enable
sntp server "0.aerohive.pool.ntp.org" priority 3
stack
member 1 4    ! N1548P
exit
ip routing
interface vlan 10
ip address 10.0.10.254 255.255.255.0
exit
interface vlan 40
ip address 10.0.40.2 255.255.255.240
exit
interface vlan 500
exit
username *** password *** privilege 15 encrypted
username *** password *** privilege 15 encrypted
username *** password *** privilege 15 encrypted
ip ssh server
!
interface Gi1/0/1
description "PC1"
switchport access vlan 40
exit
!
interface Gi1/0/2
switchport access vlan 40
exit
!
interface Gi1/0/3
switchport access vlan 40
exit
!
interface Gi1/0/4
switchport access vlan 40
exit
!
interface Gi1/0/5
switchport access vlan 40
switchport trunk allowed vlan 40
exit
!
interface Gi1/0/12
switchport access vlan 10
exit
!
interface Gi1/0/13
switchport access vlan 10
exit
!
interface Gi1/0/14
switchport access vlan 10
exit
!
interface Gi1/0/15
switchport access vlan 10
exit
!
interface Gi1/0/37
description "OPNSense MGMT"
switchport access vlan 500
exit
!
interface Gi1/0/38
description "OPNSense MGMT"
switchport access vlan 500
exit
!
interface Gi1/0/44
description "Printer_Temp"
switchport access vlan 10
exit
!
interface Gi1/0/45
description "PC"
switchport access vlan 10
switchport trunk allowed vlan 10,20,30,40
exit
!
interface Gi1/0/46
description "To OPNsense"
switchport mode trunk
switchport access vlan 10
switchport trunk allowed vlan 10,20,30,40
exit
!
interface Gi1/0/47
description "CLBSERV2"
switchport access vlan 10
switchport trunk allowed vlan 1,10,20,30,40
exit
!
interface Gi1/0/48
description "CLBSERV"
switchport access vlan 10
switchport trunk allowed vlan 1,10,20,30,40
exit

```

Okay so when building the homelab I *might have* lost sight of some security implications and some ports being configured to what they shouldn't be... 

We're going to fix that.

So going forward we will be shutting down all ports not actively in use, not due to any physical security concerns but just to be inline with best practice. We will leave the OPNsense management ports configured but shutdown, that way in the event I'm testing new features and break routing to the main gateway I have a backup connection I can turn up easily (this also has a designated port where my desk will be moved to). Also, in regards to these ports, they will only ever be connected when I am actively working on the router in a temp jumper form.

As far as SNMP, I am going to explore what all the cisco has to offer vs the original dell, configurations for that will be updated at that time (in a future MOP), SSH will also be explored later once I have brought Twingate back to functional. (I love getting new tech)

We will also be removing our interface within VLAN 40, as it is not necessary. I configured it for jumping between devices in command prompt during base configuration but never removed it and I will have some friends that are not so networking inclined working inside of that network. I.E. lets not give them access to the network that actually runs my services. Lol. 

# Cisco 3850 Config

```
en
conf t
hostname HMR-CSW-01
enable secret mustanG819
username Caleb secret mustanG819
line con 0
logging synch
login local
line vty 0 4 
logging synch
login local
ip routing
lldp run
end
wr mem
```

```
vlan 10
name Primary Management
exit
!
vlan 20
name Primary Network Services
exit
!
vlan 30
name VMs
exit
!
vlan 40
name Lab MGMT
exit
!
vlan 50
name Users
exit
!
vlan 500
name Test_VLAN
exit
!
interface vlan 10
desc remote_mgmt
ip address 10.0.10.254 255.255.255.0
no shut
exit
!
int 1/0/48
desc To_OPNSense
switchport mode trunk
switchport trunk encapsulation dot1q
switchport trunk allowed vlan 10,20,30,40,50,500
switchport mode trunk
no spanning-tree portfast
shut
no shut
exit
!
int 1/0/47
desc To_HMR-Serv-3
switchport mode trunk
switchport trunk encapsulation dot1q
switchport trunk allowed vlan <VLAN ID>,<VLAN ID>
no spanning-tree portfast
shut
no shut
exit
!
int 1/0/46
desc To_HMR-Serv-2
switchport mode trunk
switchport trunk encapsulation dot1q
switchport trunk allowed vlan <VLAN ID>,<VLAN ID>
no spanning-tree portfast
shut
no shut
exit
!
int 1/0/45
desc To_HMR-Serv-1
switchport mode trunk
switchport trunk encapsulation dot1q
switchport trunk allowed vlan <VLAN ID>,<VLAN ID>
no spanning-tree portfast
shut
no shut
exit
!
interface g1/0/1
description To_CLB-RTR-1
switchport mode access
switchport access vlan 40
spanning-tree portfast
!
interface g1/0/2
description To_CLB-RTR-2
switchport mode access
switchport access vlan 40
spanning-tree portfast
!
interface g1/0/3
description To_CLB-RTR-3
switchport mode access
switchport access vlan 40
spanning-tree portfast
!
interface g1/0/4
description To_CLB-RTR-4
switchport mode access
switchport access vlan 40
spanning-tree portfast
!
interface g1/0/44
description To_CLB-Desktop
switchport mode access
switchport access vlan 10
spanning-tree portfast
!
interface range g1/0/4 - 43
shutdown
!
end
!
```

At this point we will run our basic test commands:

```
show vlan brief
show ip int b
show int trunk
show int status
```

As this is a MOP for myself I'm not going to paste expected outputs, but I will update the document with actual outputs upon completion :)