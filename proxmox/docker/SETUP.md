# Docker Installation
Table of contents

1. <a href="#1-downloading-ct-template">Downloading CT-Template</a>
2. <a href="#2-creating-ct">Creating CT</a>
    * <a href="#ct-settingvalues">CT Settingvalues</a>
3. <a href="#3-post-create">Post-Create</a>
4. <a href="#4-start-the-ct-and-install-docker">Start the CT and Install Docker</a>
5. <a href="#5-installation-preparation-for-dockge">Installation preparation for Dockge</a>
6. <a href="#6-configuring-trunks">Configuring Trunks</a>
7. <a href="#7-network-for-docker-vlans">Network for Docker (VLAN's)</a>
    * <a href="#network-device-settingvalues">Network Device Settingvalues</a>

<br><br>


## 1. Downloading CT-Template
I used Debian Bullseye as the base template because Proxmox was already running on it and<br>
so you have the same operating system everywhere. To get the CT template, do the following.

* Go to `Local > CT Templates` in the Proxmox WebUI
* Click on the button `Templates`
* Choose and downloaded "Debian-11-Standard"
    * Decription: Debian 11 Bullseye (standard)
* Wait until it says “TASK OK” and close the window

<br><br>

## 2. Creating CT
Start by creating a new CT in proxmox WebUI.<br>
Click on the `Create CT` button above and set the CT with the following values.
I left all values that were not specified as they were or were not specified at the time of creation.

> [!NOTE]
> Activate Advanced Mode at the bottom of the window

<br>

### CT Settingvalues
* General:
    * Node:             proxmox
    * CT ID:            101
    * Name:             Dockersoft
    * Password:         defined a password I liked to use
    * confirm Password: confirm password
    * SSH public key:   Select your just downloaded template


* Template:
    * Storage:          local
    * Template:         Choose your 


* Disk:
    * Disk size:        350 GB (depends on the amount of docker container, you can go smaller and resize it later)


* CPU:
    * Cores:            3-open (I assigned most of my cores to the CT because I plan to run most of the services as docker containers)


* Memory:
    * RAM:              40 GB
    * SWAP:              6 GB

> [!NOTE]
> You must enter this value in MiB, i.e. 40.96 MiB for 40GB or 6144 MiB for 6 GB

<br>

* Network:
    * Bridge:           vmbr1 (VLAN-Network)
    * VLAN Tag:         101 (The same as my container ID)
    * IPv4:             Static
    * IPv4/CIDR:        10.1.1.2/24
    * Gateway (IPv4):   10.1.1.1 (You have to create this VLAN in OPNSense look at the [OPNSense Setup.md](../opnsense/SETUP.md))

* DNS:
    * DNS domain:       /
    * DNS servers:      8.8.8.8

After creating the CT we will add the vlan network aswell.
Now confirm the whole thing in the `Confirm area` and click on the `finish` button.

<br><br>

## 3. Post-Create
I also enabled `Start at Boot` in the Options of the CT.

<br><br>

## 4. Start the CT and Install Docker
At first create a new VLAN for this docker.<br>
My Example ID is now: 101
Use the [OPNsense documentation](../opnsense/SETUP.md#6-adding-vlans) for this and access to the internet.<br>

> [!WARNING]>
> You need to make sure the container has access to the internet.
> You can test this by pinging `ping 1.1.1.1` for example.
> Possibly you would have to turn the firewall back on.

To install docker into the CT you need to startup the container and login.<br>
To do this we simply use the get Docker script.<br>
* Run `apt update` and `apt upgrade`
* Run `apt install curl -y` to install curl

Now you can install docker.
```
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh (Shows you the current installation status)
```
You can verify the installation with `docker ps`

<br><br>

## 5. Installation preparation for Dockge
Later we will install dockge as a container manager. To make things easier for later we can already create all files for dockge.<br>
For example what I like to do is create `/opt/stacks` for dockge with `mkdir /opt/stacks`.<br>

I also like to add it to my .bashrc with `nano ~/.bashrc` and adding `cd /opt/stacks`.

<br><br>

## 6. Configuring Trunks
You will have to add a new trunk each time you want to connect a new VLAN to the Docker CT.<br>
For that its simmelar to the OPNSense setup.<br>
run the Commands in the proxmox-shell. You can edit the config with `vim` or `nano`.
```
cd /etc/pve/lxc
nano 101.conf
```

We will need to edit the line starting with `net1:` and add all your VLANS as trunks `trunks=102;103;...`<br>
It should look like this after that
```
net1: name=vlan0,bridge=vmbr1,firewall=1,hwaddr=BC:24:11:1C:45:19,type=veth,trunks=102;103;104;110;111
```
I added 102-104 and 110, 111 for now, because I will need them for sure.

> [!CAUTION]
> **MAKE SURE TO ADD NEW VLANS TO TRUNKS WHEN NEEDING A NEW VLAN**

<br><br>

## 7. Network for Docker (VLAN's)
We will also need to setup a VLAN Trunk.<br>
That way we can later seperate all docker containers into their VLAN's.<br>
For that we will need to add a new Network in the Proxmox WebUI.

* Go into your proxmoy WebUI via `yourip:8006`
* Go to "Proxmox > Docker CT > Network"
* Click on "Add" and choose "Network Device" to create a new network device

<br>

### Network Device Settingvalues
* Name:     vlan0 (way the vlans will later be named vlan0.102 internally for example. The same as in OPNSense)
* Bridge:   vmbr1 (VLAN-Network)
