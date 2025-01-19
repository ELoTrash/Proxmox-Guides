# OPNSense Guide: 
This is a guide for installing OPNSense on your Odroid H3 or H3+ device to use it as a router. 
I have been tinkering with the Odroid H3+ to use it as low power high performance router & firewall device. 

The Odroid H3+ has onboard Realtek NIC's (2x 2.5g) and has a netcard option for a total of 6x 2.5gbe NIC's making it a good option for a home router.\
The problem that users (and myself) will experience is OPNSense and PFSense use BSD, which does not contain a driver for the Realtek NIC's.\
If you install OPNSense or PFSense directly on the Odroid H3+ you will have no network interfaces to work with until you get the driver installed.\
During my testing, even with the driver installed there were a few bugs with the NIC that caused them to drop out intermittently.

## Disclaimer:
This guide is for use as is and is free to distribute, I am in no way responsible for anything that goes wrong.

## Requirements: 
1. Your Odroid must be flashed to support the netcard with the netcard bios. 
You can find more information about the BIOS and flashing guide on the official [Odroid Wiki](https://wiki.odroid.com/odroid-h3/hardware/h3_bios_update). 
2. Proxmox Installed on the Odroid H3 or H3+ device.
3. Netcard must be installed, Proxmox will use a physical ethernet port and the OPNSense or PFSense router will use two physical ethernet ports for WAN & LAN

# Steps: 
1. With Proxmox installed on the Odroid H3 or H3+, Navigate to the Network tab. 
![alt text](/Images/Odroid-H3+/OPNSense-Screenshots/Proxmox-Networking.png)
2. Take list of your Odroid Interfaces. 
Out of the box Proxmox will have 1 Network Device per physical ethernet port on your Odroid and a single Linux Bridge. (Total of 6 with Netcard installed)
![alt text](/Images/Odroid-H3+/OPNSense-Screenshots/default%20proxmox.png)
3. Create a Linux Bridge for your OPN/PFSense WAN port. 
You will only need to use the physical Port name and check the VLAN aware box. 
![alt text](/Images/Odroid-H3+/OPNSense-Screenshots/WAN%20Linux%20Bridge.png)
4. Create a Linux Bridge for your OPN/PFSense LAN port. 
You will only need to use the physical Port name and check the VLAN aware box.
![alt text](/Images/Odroid-H3+/OPNSense-Screenshots/LAN%20Linux%20Bridge.png)

When completed your networking should look similiar to this: 
![alt text](/Images/Odroid-H3+/OPNSense-Screenshots/Reference%20networking%20setup.png)
5. Download the OPNSense or PFSense ISO image to your machine.   
6. Upload the ISO image to your Proxmox storage.
![alt text](/Images/Odroid-H3+/OPNSense-Screenshots/proxmox%20upload%20iso.png)\
7. Create a Proxmox VM\
Since this will be my main router I have enabled start at boot, when my Odroid H3+ boots into Proxmox it will automatically start my router. 
![alt text](/Images/Odroid-H3+/OPNSense-Screenshots/Proxmox%20Create%20a%20VM%201.png)\
8. Select the OPNSense/PFSense image you uploaded as the OS to use.\
![alt text](/Images/Odroid-H3+/OPNSense-Screenshots/Proxmox%20Create%20a%20VM%202.png)\
9. Check the QEMU Agent box in the System tab.\
![alt text](/Images/Odroid-H3+/OPNSense-Screenshots/Proxmox%20Create%20a%20VM%203.png)\
10. Disks, I used the default sized disk with a 32gb size as this was more than enough for my router.\
![alt text](/Images/Odroid-H3+/OPNSense-Screenshots/Proxmox%20Create%20a%20VM%204.png)\
11. Configure the CPU's the router can use.\ 
My H3+ has 4 cores and 4 threads, I gave OPNSense 3 cores since I wanted to use Suricata for IDS/IPS purposes.\
![alt text](/Images/Odroid-H3+/OPNSense-Screenshots/Proxmox%20Create%20a%20VM%205.png)\
12. Dedicate the amount of memory the VM will have.\
My OPNSense has 8GB of ram assigned to it, OPNSense can easily be ran on 4GB with less features enabled.\
![alt text](/Images/Odroid-H3+/OPNSense-Screenshots/Proxmox%20Create%20a%20VM%206.png)\
13. Networking, for now Select No network device.\
![alt text](/Images/Odroid-H3+/OPNSense-Screenshots/Proxmox%20Create%20a%20VM%207.png)\
14. Select finish and create the VM.\
15. Navigate to the newly created VM's Hardware tab.\
![alt text](/Images/Odroid-H3+/OPNSense-Screenshots/proxmox%20vm%20hardware.png)\
16. In the hardware section, add a network device.\
![alt text](/Images/Odroid-H3+/OPNSense-Screenshots/add%20network%20device.png)\
17. Select the LAN and uncheck the firewall box.\
![alt text](/Images/Odroid-H3+/OPNSense-Screenshots/proxmox%20LAN%20nic.png)\
18. Repeat the same for the WAN.\
![alt text](/Images/Odroid-H3+/OPNSense-Screenshots/proxmox%20WAN%20nic.png)\
19. Start the VM and follow the OPNSense or PFSense installer guide. 

## Notes: 
- when creating the NIC the disconnect button can be enabled, inside PFSense or OPNSense you can push a to automatically start detecting new devices and then disabling the disconnect option on the nic. This will allow OPNSense or PFSense to assign the WAN or LAN ports easily during configuration. 
- `enp3s0` is my physical LAN port on the odroid h3+ netcard I have. `enp4s0` is my physical WAN port on the Odroid h3+ netcard.
