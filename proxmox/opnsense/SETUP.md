# OPNsense Installation
Table of contents

1. <a href="#1-downloading-opnsense-iso">Downloading OPNsense ISO</a>
2. <a href="#2-creating-vm-virtual-maschine">Creating VM (Virtual Maschine)</a>
    * <a href="#vm-settingvalues">VM Settingvalues</a>

## 1. Downloading OPNsense ISO
* Go to `https://opnsense.org/download/`
* Select your System architecture (my architecture: amd64)
* Select the image type to `dvd`
* Search your nearest and the best Mirror Location (my location: Germany/LeaseWeb)
* Make a right click on the Download-button and copy the Link-Address

<br>

Now follow the steps below to download and unzip the ISO file
You can now carry out the following steps from your proxmox shell which you can access in the proxmox webui
```
cd /var/lib/vz/template/iso
wget <LINK_FROM_YOUR_CLIPBOARD>
bzip2 -d <YOUR_FILENAME.bz2>
```
If the archive with the ending .bz2 still exists, you should delete it
```
rm -r <YOUR_FILENAME.bz2> 
```
<br><br><br>


## 2. Creating VM (Virtual Maschine)
Start by creating a new VM in proxmox.<br>
I used VM ID 100 to display it at the top because its the first entry point for every request to the server.<br>
Later I also assign VLAN100 to the opnsense.

Click on the `Create VM` button above and set the VM with the following values.
I left all values that were not specified as they were or were not specified at the time of creation.

> [!NOTE]
> Activate Advanced Mode at the bottom of the window

### VM Settingvalues
* General:
    * Node:             proxmox
    * VM ID:            100
    * Name:             OPNsense
<br><br>
* OS:
    * ISO image:        Choose your iso here
    * Qemu Agent:       True (That way proxmox qemu can tell the vm to start, stop and restart when the qemu agent is installed)
<br><br>
* Disk:
    * Disk size:        10-30 GB (depends on the amount of firewall rules, configs and plugins)
<br><br>
* CPU:
    * Cores:            2-4 (depends on the network traffic)
    * Extra CPU Flags:  aes > on
<br><br>
* Memory:
    * 3GB-6GB (depends on the network traffic and rules and plugins)
> [!NOTE]
> You must enter this value in MiB, i.e. 8192 MiB for 4GB
<br><br>
* Network:
    * Bridge:            vmbr0 (WAN)
    * Multiqueue:        8 (For better performance)

After creating the VM we will add the vlan network aswell.
Now confirm the whole thing in the `Confirm area` and click on the `finish` button.
<br><br><br>


