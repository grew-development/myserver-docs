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

**Exaple:**<br>
´´´
HOSTNAME proxmox.grew-development.de
´´´<br>

I also changed the disk size, because by default the config does not set the partition sized right. I assigned as much as possible and also a bit of swap.

**Exaple:**<br>
´´´
LV  vg0 root    /       ext3    460GiB
LV  vg0 swap    swap    ext3    12GiB 
´´´<br>

Also confirm the next two confirmations.

If you follow this the `installimage` script will automatically install proxmox with RAID. And will setup all basic configs to get started on its own.
After installation the server will restart and proxmox will then be accessable via port 8006 on your IP4-Address.



## 2. Setting Up Ethernet Interfaces and Switches

