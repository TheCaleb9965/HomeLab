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

VMs (To be added after current troubleshooting session)
