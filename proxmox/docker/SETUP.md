# Docker Installation
Table of contents

1. <a href="#1-downloading-ct-template">Downloading CT-Template</a>
2. <a href="#2-creating-ct">Creating CT</a>
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


