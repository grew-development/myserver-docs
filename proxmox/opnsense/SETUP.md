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
5. <a href="#5-post-install">Post-Install</a>
    * <a href="#removing-install-iso">Removing Install ISO</a>
    * <a href="#configuring-wan-for-opnsense">Configuring WAN for OPNsense</a>
    * <a href="#accessing-webui">Accessing WebUI</a>
        * <a href="#via-powershell">Via PowerShell</a>
        * <a href="#via-putty">Via Putty</a>
    * <a href="#running-and-setup-wizzard">Running and Setup Wizzard</a>

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


## 5. Post-Install
### Removing Install ISO
After the installation I like to stop the VM and remove the install ISO<br>
so that it will not accidential run again at some point or time.

Currently we can't just stop the VM in proxmox (because of the missing qemu-agent).<br>
So you will have to wait till opnsense starts to login (as root and with your currently specified password) and<br>
shutdown from the console. You can do so by choosing option 5.

After that, you can go to the hardware and remove the installation ISO from the CD/DVD Drive and<br>
then restart the VM for the next steps.

### Configuring WAN for OPNsense
By default Opnsense will try to setup the WAN with some default values.<br>
But these will not match our configuration and sometimes it will even choose the wrong network adapter.

First make sure you are logged in.<br>
You will then have the option 1 to assign the interfaces. Choose option 1: `1) Assign interfaces`
I use the following values within the setting:

* LAGS: No
* VLAN: No (not now)
* WAN-Interface: vtnet0
* LAN-Interface: / (nothing for now just hit enter)
* Optional-Interface: / (nothing hit enter)
* Proceed: Yes

Next we will have to tell opnsense the config of the WAN.<br>
Choose option 2 to set interface IP address: `2) Set interface IP address`
I use the following values within the setting:

* IPv4 DHCP: No
* IPv4: 10.10.10.1 (as configured in interfaces in proxmox)
* Subnet: 31 (only 10.10.10.0 and 10.10.10.1)
* WAN-Gateway: 10.10.10.0
* Namesevers: No
* IPv4 Nameserver: 1.1.1.1 (Cloudflare) or 8.8.8.8 (Google)
* DHCP IPv6: No
* IPv6: / (none press enter)
* DHCP WAN: No
* WebUI HTTPS -> HTTP: No (HTTPS better)
* Certificate: Yes (self signed tho)
* Restore defaults: Yes

### Accessing WebUI
You successfully configured opnsense.<br>
You can try to access it under `https://yourip` but<br>
you will see that the login will not work because the IP is not whitelisted.

You can access it via a ssh tunnel tho.<br>
`ssh -L 443:10.10.10.1:443 root@yourip` You will then be able to access it with https://localhost on your machine and login into opnsense.

#### Via PowerShell
Open PowerShell (not as Administrator) and enter the following command: 
```
ssh -L 443:10.10.10.1:443 root@yourip -i <PATH_TO_YOUR_SSH_KEY>
```

If you receive the error message **"WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!"** then you have to carry out the following intermediate steps to continue.

1. go to your Explorer on the following path `C:\\Users\\<YOUR_USERNAME>\.ssh\`
2. Open the file `knwon_hosts` and delete the line with your IP Address
3. Safe the File and return to the PowerShell 

Now do the previous step again. Enter the following command:
```
ssh -L 443:10.10.10.1:443 root@yourip -i <PATH_TO_YOUR_SSH_KEY>
```

You will be asked if you are sure you want to connect.<br>
Answer this question with yes

```
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
```

#### Via Putty
Open Putty.exe and type your IP address. Before connecting, add the tunnel we need.
* Go to Connection > SSH > Tunnels
* Now enter the following
    * Source port:  443
    * Destination:  10.10.10.1:443
* Then press `Add` and connect
> [!TIP]
> I did the same for HTTP too.<br>
> i.e. port 80 added.

### Running and Setup Wizzard
If you have entered everything correctly, you can now connect via `https://localhost` in your Browser. To setup everything important I recommend running the setup wizard. It will configure some important things needed for further configuration

Logged in with `root`and your personal password.

If it doesn't open automatically go to "System > Wizard"

I would suggest changing the domain to something like: `opnsense.yourdomain.com`.<br>
I would also set a second dns server like 8.8.8.8 (google) for example.

In the next step you can set your timezone.

On the next page you will not need to change anything.<br>
You can let it stay as is.

On the next page you can also just hit next, because we will configure vlans not LAN.

On the next page you can change your root password again if you want to.

After the wizzard finishes it will reload.

