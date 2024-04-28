# Server Installation

## 0. Start the Rescue-Mode
For the first step of my (new) installation I use the `Rescue-Mode` from [Hetzner.com](https://hetzner.com).<br>
Start this mode as follows:
1. Login to Hetzner and open [`https://robot.hetzner.com/server`](https://robot.hetzner.com/server)
2. Select your Public-SSh-Key and activate the mode by clicking on the 'Activate Rescue System' button.
3. Reboot your server
4. Now open your Putty.exe
5. Now enter the following data
    * Hostname or IP-Address (I used my IP-Address)
    * Port 22
    * Enter your path to the private SSH key (Connection > SSH > Auth > Credentails)
6. As soon as the console is opened, it asks for a user name "Login as:". Enter the user `"root"` here
7. You received the password from Hetzner in the Rescu window. Copy it now.



## 1. Install Proxmox
To install Proxmox on my hetzner server, I used the `installimage` Script provided by hetzner.com<br>
I used the `Proxmox-Bullseye` version. Because at the time of writing this Debian Bullseye is the most stable version of debian.

As soon as the console is opened and login with "root@rescue", than write `installimage` and choose `Other > Proxmox[...]-Debian-Bullseye`

In the install config I only **changed the hostname to proxmox.mydomain.com**. I also changed the disk size, because by default the config does not set the partition sized right. I assigned as much as possible and also a bit of swap.

**Exaple:**
```
HOSTNAME proxmox.grew-development.de
```
<br>

I also changed the disk size, because by default the config does not set the partition sized right. I assigned as much as possible and also a bit of swap.

**Exaple:**
```
LV  vg0 root    /       ext3    460GiB
LV  vg0 swap    swap    ext3    12GiB 
```
<br>

Save (F2) and Quit (F10) the configuration file.
Also confirm the next two confirmations and wait until the installation is finished.

If you follow this the `installimage` script will automatically install proxmox with RAID. And will setup all basic configs to get started on its own.
After installation the server will restart and proxmox will then be accessable via port 8006 on your IP4-Address.



## 2. Setting Up Ethernet Interfaces and Switches
After restarting the server, open your browser and go to the URL `yourIP4:8006`.

> [!NOTE]
> You may have to accept the risk, as your SSL certificate is currently Self Signed and the browser just doesn't trust them by default.

But first you should set your password via SSH console.<br>
* open putty.exe and log in.
* enter `passwd` in the console.

I wanted to use VLANs on my server to be able to seperate everything as much as possible even if its maby a bit over the top. I started by configuring a vmbr0 bridge for WAN and internet access for the opnsense later. I also added a vmbr1 bridge for the VLANs later. I used a ovs bridge which you have to install first.

You can now carry out the following steps from your proxmox shell which you can access in the proxmox webui or<br>
in the Putty Console.

```
apt update
apt upgrade
apt install openvswitch-switch
```

If you want to copy my config look her [/etc/network/interfaces]. There you will find my exact config to copy.<br>
But remember the values you read beforehand<br>
You will have to adjust your ip4 and gateway to the ones given to you from proxmox.<br>
To copy this config just use `vim` or `nano`
After that you will need to restart the server.

```
nano /etc/network/interfaces -y
reboot
```

> [!CAUTION]
> MAKE SURE THE VALUES ARE CORRECT OTHERWISE YOU WILL NOT REACH THE SERVER ON THE IP ANYMORE<br>
> (HETZNER RESCUE MODE TO YOUR HELP)

### What did i configure?
All Requests to the server except on port 22(SSH) and 8006(proxmox) will get redirected to the opnsense (static IP).<br>
I only configured IPv4 you can also setup it for IPv6 if needed



## 3. Install OPNsense (my Firewall)
