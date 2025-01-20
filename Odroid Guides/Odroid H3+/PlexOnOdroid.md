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
## Initial VM configuration:
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
9. Create and start the VM.\
![alt text](/Images/Odroid-H3+/Plex/start%20vm.png)
10. Follow the steps in the No VNC console to install Ubuntu Server.\
Note the IP address, and check the box to install OpenSSH in the steps.\
![alt text](/Images/Odroid-H3+/Plex/Console%20vm.png)

## Proxmox IOMMU Configuration:
1. Open a host Proxmox shell and run the following command `nano /etc/default/grub` and add the following:
```
GRUB_DEFAULT=0
GRUB_TIMEOUT=5
GRUB_DISTRIBUTOR=`lsb_release -i -s 2> /dev/null || echo Debian`
GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on iommu=pt igfx_off video=efifb:off i915.enable_guc=3 i915.max_vfs=7"
GRUB_CMDLINE_LINUX=""
```
![alt text](/Images/Odroid-H3+/Plex/example%20etc_default_grub%20file.png)
2. Install the Git repo for the i915-sriov-dkms.\
Note: Git may need to be installed first, can be done with `apt install git`\
`git clone https://github.com/strongtz/i915-sriov-dkms.git`\
3. Navigate into the directory of the cloned file:\
`cd i915-sriov-dkms`\
![alt text](/Images/Odroid-H3+/Plex/git%20clone%20folder%20structure.png)\
4. Install the github contents:\
`dkms install -m i915-sriov-dkms -v 2024.08.09 --force`\
5. Install the proxmox headers:\
`apt install proxmox-headers-6.8.8-2-pve proxmox-kernel-6.8.12-1-pve`
6. Update the grub with the following command: 
`update-grub`\
7. Restart your Proxmox server.

## IGPU Passthrough: 
1. Back at the Ubuntu VM, change the Display to `none`.\
![alt text](/Images/Odroid-H3+/Plex/display%20none.png)
2. Add a PCIE device for the GPU, it will be labeled as a UHD graphics for the Odroid H3+.\
![alt text](/Images/Odroid-H3+/Plex/gpu%20passthrough.png)
3. Start the VM and SSH to it with a tool such as Putty. 
4. Install the latest updates with `sudo apt update && sudo apt upgrade -y`. 
5. Install plex with the following:
```
echo deb https://downloads.plex.tv/repo/deb public main | sudo tee /etc/apt/sources.list.d/plexmediaserver.list
curl https://downloads.plex.tv/plex-keys/PlexSign.key | sudo apt-key add -
sudo apt update
sudo apt install plexmediaserver
```
6. Install the Intel GPU Tools:
`sudo apt install intel-gpu-tools`
To use the tools run `sudo intel_gpu_top` this will show you the utilization of your Intel IGPU. 
7. Next we will create a Ramdisk for Plex to use for transcoding, run the following command:
`sudo mkdir /transcode`
8. Open the FSTAB file, `sudo nano /etc/fstab` 
Add the following to the fstab file: 
```
`tmpfs /transcode tmpfs rw,nodev,nosuid,noexec,nodiratime,size=8G 0 0 `
```
![alt text](/Images/Odroid-H3+/Plex/fstab%20file.png)
9. Save the file and run `sudo mount -a` 

## Configuring Plex: 
1. Navigate to the following in a web browser `https://ubuntuIP:32400/web`. 
2. Sign in and link your Plex to your new server. 
3. When logged in navigate to the Plex settings (wrench icon)
4. Search for the `Transcoder` settings. 
![alt text](/Images/Odroid-H3+/Plex/plex%20transcoder%20settings.png)
5. Set the transcoder temporary directory to `/transcode`
![alt text](/Images/Odroid-H3+/Plex/ramdisk.png)
6. Scroll down and enable the `Use hardware-accelerated video encoding` 
NOTE: This setting requires a Plex pass. 
7. Select the `Intel JasperLake [UHD Graphics]` from the drop down and save changes.
![alt text](/Images/Odroid-H3+/Plex/save%20changes.png)
