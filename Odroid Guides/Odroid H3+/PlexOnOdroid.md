# Plex Guide: 
This is a guide for installing Plex on your Odroid H3 or H3+ device to use it for a media center.
I have been tinkering with the Odroid H3+ to use it as a Plex media server.\

The Odroid H3+ has an onboard Intel GPU that can be used for transcoding.\
The Intel integrated GPU (igpu) has some difficulty passing it through to a VM for transcoding media.\
The type of igpu being used requires updating the grub bootloader inside proxmox with certain parameters as well as certain VM requirements.\

## Disclaimer:
This guide is for use as is and is free to distribute, I am in no way responsible for anything that goes wrong.

## Requirements: 
1. Proxmox must be installed as the main OS/Hypervisor.\ 
2. Plex pass is required for hardware transcoding.\
3. Atleast 8gb of ram is required for the Plex VM alone.\ 