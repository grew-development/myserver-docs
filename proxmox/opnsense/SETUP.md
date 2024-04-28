# OPNsense Installation
Table of contents

1. <a href="#1-downloading-opnsense-iso">Downloading OPNsense ISO</a>
2. <a href="#2-creating-vm-virtual-maschine">Creating VM (Virtual Maschine)</a>
    * <a href="#vm-settingvalues">VM Settingvalues</a>
3. <a href="#3-post-create">Post-Create</a>
    * <a href="#network-device-settingvalues">Network Device Settingvalues</a>
    * <a href="#setup-trunks">Setup Trunks</a>
    * <a href="#setup-start-at-boot">Setup "Start at boot"</a>
4. <a href="#4-start-the-vm-and-install-opnsense">Start the VM and Install OPNsense</a>

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


## 3. Post-Create
We will need to add the vlan network now aswell.<br>
For that you will need to head to the Hardware Section of your opnsense VM in the Proxmox Web Interface.<br>
Click on "Add" and choose "Network Device" to create a new network device.

### Network Device Settingvalues
* Bridge:                  vmbr1 (VLAN-Network)
* VLAN Tag:                100 (The same as the VM ID)
* Multiqueue:              8 (For better performance)

### Setup Trunks
Now we also need to tell Proxmox, that Opnsense acts as a trunk in the VLAN-NET. (If you don't know what a trunk is look it up!)

You can now follow the steps below to edit the vm config from your Proxmox shell, accessible in the Proxmox webui.
I use `nano` to edit the files here, but you can also use `wim`.
```
cd /etc/pve/qemu-server
nano 100.conf
```
Go to the line of net1 and add `trunks=1-4095`. The line should then look something like this:
```
net1: virtio=92:39:CF:F0:F9:A8,bridge=vmbr1,firewall=1,queues=8,tag=100,trunks=1-4095
```

### Setup "Start at boot"
As a final step, if you didn't specify it during installation, you'll want to make sure that "Start at boot" is checked in the VM options.<br>
Otherwise, opnsense will not start when the server restarts.
<br><br><br>


## 4. Start the VM and Install OPNsense
If you did everything correct you should be able to startup the VM.<br>
Go to the `console-tab` to view the screen of opnsense.<br>
Click “Start Now”

Wait till the opnsense installer promst you with a login.

You should then be able to login as `installer` and password `opnsense`.

Start by configuring your keymap. And then continue.<br>
I use the German-Keymap.

Next you have to choose how you want to install opnsense.<br>
I used ZFS but you can choose what you like.

After that I used the stripe installer,<br>
because my proxmox has already raid so I don't need raid here anymore.

Next select your drive with spacebar and press enter and choose Yes to install opnsense.<br>
Here you will be asked if you need to be sure that the installation is required. **Confirm this with yes**

After that you can choose a secure root password.<br>
**Remember that password!**

Then you can finish the installation.
<br><br><br>


