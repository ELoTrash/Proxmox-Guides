# Plex Guide: 
This is a guide for installing Plex on your Odroid H3 or H3+ device to use it for a media center.
I have been tinkering with the Odroid H3+ to use it as a Plex media server.

The Odroid H3+ has an onboard Intel GPU that can be used for transcoding.
The Intel integrated GPU (igpu) has some difficulty passing it through to a VM for transcoding media.
The type of igpu being used requires updating the grub bootloader inside proxmox with certain parameters as well as certain VM requirements.

## Disclaimer:
This guide is for use as is and is free to distribute, I am in no way responsible for anything that goes wrong.

## Requirements: 
1. Proxmox must be installed as the main OS/Hypervisor. 
2. Plex pass is required for hardware transcoding.
3. Atleast 8gb of ram is required for the Plex VM alone (I used 12).
4. Putty or some SSH tool must be used to SSH into the VM since it will have no display output.
5. PCIE pass through enabled on Proxmox. [Proxmox Documentation](https://pve.proxmox.com/wiki/PCI(e)_Passthrough). 

# Steps: 
1. With Proxmox installed on the Odroid H3 or H3+, Create a VM.\
![alt text](/Images/Odroid-H3+/Plex/Create%20a%20VM.png)
2. Insert a VM name and select the start at boot option.\
![alt text](/Images/Odroid-H3+/Plex/Create%20a%20VM%202.png)
3. Select the OS, I am using Ubuntu Server.\
Note: you will not be able to use a GUI or VNC interface once the IGPU is passed through.\
![alt text](/Images/Odroid-H3+/Plex/Create%20a%20VM%203.png)\
4. Change the machine and bios.\
![alt text](/Images/Odroid-H3+/Plex/Create%20a%20VM%204.png)\
5. Add a storage disk, I used 64gb for my disk as I it will only be used for the Ubuntu OS in my case.\
![alt text](/Images/Odroid-H3+/Plex/Create%20a%20VM%205.png)\
6. Assign a CPU, I assigned 2 cores to my Plex server as the IGPU will do the heavy lifting. 
![alt text](/Images/Odroid-H3+/Plex/Create%20a%20VM%206.png)\
7. Add 12GB of RAM to the VM.\
We will assign 8GB of RAM to a ramdisk to be used for transcoding. Transcoding requires a lot of writing and deleting of data, which can be harmful to SSD's and slow on HDD's.\
![alt text](/Images/Odroid-H3+/Plex/Create%20a%20VM%207.png)\
8. Network, I left this as default with the `vmbr0` bridge that proxmox uses for the UI (if you are setting this up after following my OPNSense guide this will make more senes).\
![alt text](/Images/Odroid-H3+/Plex/Create%20a%20VM%208.png)\
9. 