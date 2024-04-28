# Docker Installation
Table of contents

1. <a href="#1-downloading-ct-template">Downloading CT-Template</a>
2. <a href="#2-creating-ct">Creating CT</a>
    * <a href="#ct-settingvalues">CT Settingvalues</a>
3. <a href="#3-post-create">Post-Create</a>
4. <a href="#4-start-the-ct-and-install-docker">Start the CT and Install Docker</a>
<br><br><br>


## 1. Downloading CT-Template
I used Debian Bullseye as the base template because Proxmox was already running on it and<br>
so you have the same operating system everywhere. To get the CT template, do the following.

* Go to `Local > CT Templates` in the Proxmox WebUI
* Click on the button `Templates`
* Choose and downloaded "Debian-11-Standard"
    * Decription: Debian 11 Bullseye (standard)
* Wait until it says “TASK OK” and close the window
<br><br><br>


## 2. Creating CT
Start by creating a new CT in proxmox WebUI.<br>
Click on the `Create CT` button above and set the CT with the following values.
I left all values that were not specified as they were or were not specified at the time of creation.

> [!NOTE]
> Activate Advanced Mode at the bottom of the window
<br><br>

### CT Settingvalues
* General:
    * Node:             proxmox
    * CT ID:            101
    * Name:             Dockersoft
    * Password:         defined a password I liked to use
    * confirm Password: confirm password
    * SSH public key:   Select your just downloaded template
<br><br>
* Template:
    * Storage:          local
    * Template:         Choose your 
<br><br>
* Disk:
    * Disk size:        350 GB (depends on the amount of docker container, you can go smaller and resize it later)
<br><br>
* CPU:
    * Cores:            3-open (I assigned most of my cores to the CT because I plan to run most of the services as docker containers)
<br><br>
* Memory:
    * RAM:              40 GB
    * SWAP:              6 GB
> [!NOTE]
> You must enter this value in MiB, i.e. 40.96 MiB for 40GB or 6144 MiB for 6 GB
<br><br>
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
<br><br><br>

### 3. Post-Create
I also enabled `Start at Boot` in the Options of the CT.
<br><br><br>


### 4. Start the CT and Install Docker
> [!WARNING]
> At first create a new VLAN for this docker.<br>
> My Example ID is now: 101
> Use the [OPNsense documentation](../opnsense/SETUP.md#6-adding-vlans) for this and access to the internet.<br>
>
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
