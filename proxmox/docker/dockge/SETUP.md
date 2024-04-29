# Dockge Installation
Table of contents

1. <a href="#1-create-the-docker">Create the DockerCompose-File</a>
2. <a href="#2-create-a-new-vlan-in-opnsense">Create a new VLAN in OPNsense</a>
3. <a href="#3-start-container">Start container</a>
4. <a href="#4-create-a-subdomain-with-the-provider">Create a subdomain with the provider</a>
5. <a href="#5-create-a-proxy-dockgeyourdomainde">Create a Proxy "dockge.yourdomain.de"</a>
    * <a href="#allowing-access-from-proxy-to-dockge">Allowing Access from proxy to dockge</a>
    * <a href="#add-proxy-dockgeyourdomainde">Add Proxy (dockge.yourdomain.de)</a>
6. <a href="#6-login-into-webinterface">Login into webinterface</a>
7. <a href="#7-adding-domain-to-dockge-webui-optional">Adding Domain to Dockge WebUI (Optional)</a>


<br><br>

## 1. Create the DockerCompose-File
We create a new folder called “dockge” in /opt/stacks.

```
mkdir /opt/stacks/dockge
```
We will now create our docker-compose.yml file in it. We open this straight away with the command “nano” or “vim”
```
nano docker-compose.yml
```
You can see what is written there from my <a href="./docker-compose.yml">docker compose file</a>.<br>
I have also written explanations there as a comment for the individual points that you can change.

<br><br>

## 2. Create a new VLAN in OPNsense
You should still be connected via the domain `https://opnsense.yourdomain.de`.<br>
Then create a new VLAN with the OPNsense documentation
* <a href="/proxmox/opnsense/SETUP.md#create-a-new-vlan-generel-instructions">Create a VLAN</a>
* <a href="/proxmox/opnsense/SETUP.md#adding-and-enable-of-the-new-interface-generel-instructions">Adding and enable of the new interface</a>
* <a href="/proxmox/opnsense/SETUP.md#add-the-rule-rfc1918">Add the Rule RFC1918</a>

The VLAN-Values is like this:
```
Device: vlan0.103
Parent: vtnet1
VLAN tag: 103
Descript: 103_Dockge
```

<br><br>

## 3. Start container
If you are no longer in the folder you previously created, go into the folder
```
cd /opt/stacks/dockge
```
To start the container now, use the following command
```
docker compose up -d
```
>[!NOTE]
>**Explanation of the command**<br>
>`docker` is the app<br>
>`compose` you address the container<br>
>`up` you start it<br>
>`-d` means that you do it covertly as detached

<br>

>[!NOTE]
>If you want to know which container is currently active in docker, use Command:
>`docker ps`

>[!NOTE]
>If you want to see the logs for a container, go to the folder and run the command:
>`docker compose logs`

<br><br>

## 4. Create a subdomain with the provider
In order to access this website, I first create an A record with my domain provider.<br>
Create a DNS record for the subdomain `dockge.yourdomain.de` as described in the <a href="/proxmox/opnsense/SETUP.md#create-dns-record">OPNsense documentation</a>

<br><br>

## 5. Create a Proxy "dockge.yourdomain.de"
### Allowing Access from proxy to dockge
* Open `https://opnsense.yourdomain.de`
* Go to "Firewall > NAT > Port Forward"
* Create a new NAT rule
    * Interface:                Proxy VLAN
    * Destination:              Dockge-Alias (10.1.3.2)
    * Destination Port from:    (other) 5001 - Your Dockge WebPort
    * Destination Port to:      (other) 5001 - Your Dockge WebPort
    * Redirect Target IP:       Dockge-Alias (10.1.3.2)
    * Redirect Target Port:     (other) 5001 - Your Dockge WebPort
* Safe

<br>

### Add Proxy (dockge.yourdomain.de)
* Open `https://proxy.yourdomain.de`
* Go to "Hosts > Proxy Hosts"
* Click green "Add Proxy Host"-Button
* Now enter the following values
    * Details-Tab
        * Domainname:               dockge.yourdomain.de
        * Scheme:                   http
        * Forward Hostname / IP:    10.1.3.2
        * Forward Port:             5001
        * Websockets Support:       True
    * SSL-Tab
        * SSL Certificate:          "Requet a new SSL Certificalte"
        * Force SSL:                True
        * HTTP/2 Support:           True
        * Email Address for LE:     Your E-Mailadress
        * I Agree to the LE ToS:    True
* Click Safe-button

It will now take a while as the SSL certificate is created.

<br><br>

## 6. Login into webinterface
* Open `https://dockge.yourdomain.de`
* Please create your admin Account
    * Username
    * Password
* Save 

Congratulations. You can now access your Dockge-System via `dockge.yourdomain.de`.<br>
Now you can use your Dockge-System.

<br><br>

## 7. Adding Domain to Dockge WebUI (Optional)
* Open `https://dockge.yourdomain.de`
* Click on an compose, of the left side
* Click edit
* Add this Line at the end of the docker-compose.yml
```
x-dockge:
  urls:
    - ${NAMEOFCONTAINER_EXTERNAL_IP}
```
* Write the following lines in the .env-file
```
NAMEOFCONTAINER_EXTERNAL_IP="https://YOUR_DOMAIN.de"
```
* Click at the top "Deployen"

>[!IMPORTANT]
>If you edit and update the docker-compose docker in Dockge,<br>
>then you must start this container manually
