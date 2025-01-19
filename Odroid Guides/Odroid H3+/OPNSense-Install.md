# OPNSense Guide: 
This is a guide for installing OPNSense on your Odroid H3 or H3+ device to use it as a router. 
I have been tinkering with the Odroid H3+ to use it as low power high performance router & firewall device. 

The Odroid H3+ has onboard Realtek NIC's (2x 2.5g) and has a netcard option for a total of 6x 2.5gbe NIC's making it a good option for a home router. 
The problem that users (and myself) will experience is OPNSense and PFSense use BSD, which does not contain a driver for the Realtek NIC's. 
If you install OPNSense or PFSense directly on the Odroid H3+ you will have no network interfaces to work with until you get the driver installed. 
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
![alt text](/Images/Odroid-H3+/OPNSense-Screenshots/WAN%20Linux%20Bridge.png
4. Create a Linux Bridge for your OPN/PFSense LAN port. 
You will only need to use the physical Port name and check the VLAN aware box.
![alt text](/Images/Odroid-H3+/OPNSense-Screenshots/LAN%20Linux%20Bridge.png

