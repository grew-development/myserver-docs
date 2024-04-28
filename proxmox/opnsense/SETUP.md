# OPNsense Installation
Table of contents

1. <a href="#1-downloading-opnsense-iso">Downloading OPNsense ISO</a>

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

