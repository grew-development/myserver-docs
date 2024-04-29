# Proxy Installation
Table of contents

1. <a href="#1-create-the-docker">Create the DockerCompose-File</a>
2. <a href="#2-create-a-new-vlan-in-opnsense">Create a new VLAN in OPNsense</a>
3. <a href="#3-start-container">Start container</a>
4. <a href="#4-connect-to-the-web-interface">Connect to the web interface</a>
    * <a href="#create-temporary-firewall-rule">Create temporary firewall rule</a>
5. <a href="#5-login-into-webinterface">Login into webinterface</a>
6. <a href="#6-create-a-subdomain-with-the-provider">Create a subdomain with the provider</a>
7. <a href="#7-create-a-proxy-proxyyourdomainde">Create a Proxy "proxy.yourdomain.de"</a>
    * <a href="#allowing-access-from-wan-to-proxy">Allowing Access from wan to proxy</a>
    * <a href="#add-proxy-proxyyourdomainde">Add Proxy (proxy.yourdomain.de)</a>
8. <a href="#8-create-a-proxy-opnsensegrew-developmentde">Create a Proxy "opnsense.yourdomain.de"</a>
    * <a href="#allowing-access-from-proxy-to-opnsense">Allowing Access from proxy to opnsense</a>
    * <a href="#add-proxy-opnsenseyourdomainde">Add Proxy (opnsense.yourdomain.de)</a>
9. <a href="#9-create-a-proxy-proxmoxgrew-developmentde">Create a Proxy "proxmox.yourdomain.de"</a>
    * <a href="#allowing-access-from-proxy-to-proxmox">Allowing Access from proxy to proxmox</a>
    * <a href="#add-proxy-proxmoxyourdomainde">Add Proxy (proxmox.yourdomain.de)</a>

<br><br>

## 1. Create the DockerCompose-File
We create a new folder called “nginx-proxy-manager” in /opt/stacks.

```
mkdir /opt/stacks/nginx-proxy-manager
```
We will now create our docker-compose.yml file in it. We open this straight away with the command “nano” or “vin”
```
nano docker-compose.yml
```
You can see what is written there from my <a href="./docker-compose.yml">docker compose file</a>.<br>
I have also written explanations there as a comment for the individual points that you can change.

<br><br>

## 2. Create a new VLAN in OPNsense
You should still be connected via the domain opnsense.yourdomain.de:9443 and a deactivated firewall in the Proxmox WebUI.<br>
If not, please take another look at the OPNsense documentation on the topic: <a href="/proxmox/opnsense/SETUP.md#7-configurate-the-access-about-opnsenseyourdomainde">"Configurate the Access about opnsense.yourdomain.de"</a>.

Then create a new VLAN with the OPNsense documentation
* <a href="/proxmox/opnsense/SETUP.md#create-a-new-vlan-generel-instructions">Create a VLAN</a>
* <a href="/proxmox/opnsense/SETUP.md#adding-and-enable-of-the-new-interface-generel-instructions">Adding and enable of the new interface</a>

The VLAN-Values is like this:
```
Device: vlan0.102
Parent: vtnet1
VLAN tag: 102
Descript: 102_Proxy
```

<br><br>

## 3. Start container
If you are no longer in the folder you previously created, go into the folder
```
cd /opt/stacks/nginx-proxy-manager
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

## 4. Connect to the web interface
### Create temporary firewall rule
We would like to create a temporary firewall rule to access the WebUI via port 81.<br>
To do this, first open opnsense via the browser: `opnsense.yourip.de`

* Go to "Firewall > NAT > Port Forward"
* Create a new NAT rule
    * Interface:                WAN
    * Destination:              WAN_ADDRESS
    * Destination Port from:    (other) 81
    * Destination Port to:      (other) 81
    * Redirect Target IP:       Proxy-Alias

Now you can access the web interface of your proxy manager via `http://yourip:81`
>[!NOTE]
>You can find all further information in the <a href="/proxmox/opnsense/SETUP.md#allowing-access-to-other-vlans--nat-generel-instructions">OPNsense setup documentation</a>.

<br><br>

## 5. Login into webinterface
Please use the following data for the first login
```
E-Mailadress: admin@example.com
PW: changeme
```
You will then be asked to change your userinformation and the password, but I recommend filling this out.<br>
Save and Take over everything

<br><br>

## 6. Create a subdomain with the provider
In order to access this website, I first create an A record with my domain provider.<br>
Now create all the other subdomains you need in the following diagram.<br>
Here is a list of my subdomains, which I am still creating in this documentation.
* proxmox.yourdomain.de
* opnsense.yourdomain.de

Create a DNS record for all subdomains as described in the <a href="/proxmox/opnsense/SETUP.md#create-dns-record">OPNsense documentation</a>

<br><br>

## 7. Create a Proxy "proxy.yourdomain.de"
### Allowing Access from wan to proxy
* Open `opnsense.yourdomain.de:9443`
* Go to "Firewall > NAT > Port Forward"
* Create a new NAT rule
    * Interface:                WAN
    * Destination:              WAN_ADDRESS
    * Destination Port from:    http (80)
    * Destination Port to:      http (80)
    * Redirect Target IP:       Proxy-Alias (10.1.2.2)
    * Redirect Target Port:     http (80)
* Safe

Repeat creating a NAT rule only with the ports HTTPS (443)

<br>

### Add Proxy (proxy.yourdomain.de)
* Open `http://yourip:81`
* Go to "Hosts > Proxy Hosts"
* Click green "Add Proxy Host"-Button
* Now enter the following values
    * Details-Tab
        * Domainname:               proxy.yourdomain.de
        * Scheme:                   http
        * Forward Hostname / IP:    127.0.0.1
        * Forward Port:             81
        * Websockets Support:       True
    * SSL-Tab
        * SSL Certificate:          "Requet a new SSL Certificalte"
        * Force SSL:                True
        * HTTP/2 Support:           True
        * Email Address for LE:     Your E-Mailadress
        * I Agree to the LE ToS:    True
* Click Safe-button

It will now take a while as the SSL certificate is created.<br>
Now delete the temporary NAT rule in OPNsense.<br>

Congratulations. You can now access your NGINX Proxy Manager via `proxy.yourdomain.de`.<br>
Now we create the proxies for the other domains.

<br><br>

## 8. Create a Proxy "opnsense.grew-development.de"
We now create access to OPNsense via a subdomain.<br>
The reason behind it is that we no longer need an SSL tunnel and we can leave the firewall permanently activated via the Proxmox WebUI<br>
in the OPNsense console and no longer have to enter `-pfctl -d`.

### Allowing Access from proxy to opnsense
* Open `opnsense.yourdomain.de:9443`
* Go to "Firewall > NAT > Port Forward"
* Create a new NAT rule
    * Interface:                Proxy VLAN
    * Destination:              PROXY_ADDRESS
    * Destination Port from:    (other) 9443 - Your Opnsense WebPort
    * Destination Port to:      (other) 9443 - Your Opnsense WebPort
    * Redirect Target IP:       127.0.0.1
    * Redirect Target Port:     (other) 9443 - Your Opnsense WebPort
* Safe

<br>

### Add Proxy (opnsense.yourdomain.de)
* Open `proxy.yourdomain.de`
* Go to "Hosts > Proxy Hosts"
* Click green "Add Proxy Host"-Button
* Now enter the following values
    * Details-Tab
        * Domainname:               opnsense.yourdomain.de
        * Scheme:                   https
        * Forward Hostname / IP:    10.1.2.1 (OPNsense Gateway)
        * Forward Port:             9443
        * Websockets Support:       True
    * SSL-Tab
        * SSL Certificate:          "Requet a new SSL Certificalte"
        * Force SSL:                True
        * HTTP/2 Support:           True
        * Email Address for LE:     Your E-Mailadress
        * I Agree to the LE ToS:    True
* Click Safe-button

It will now take a while as the SSL certificate is created.<br>

Congratulations. You can now access your OPNsense via `opnsense.yourdomain.de`.<br>
Now you could also close the SSL tunnels and access opnsense without deactivating the firewall.

<br><br>

## 9. Create a Proxy "proxmox.grew-development.de"
We now create access to Proxmox via a subdomain.<br>
Advantage: you no longer have to enter my public IPv4 address in the browser.

### Allowing Access from proxy to proxmox
* Open `opnsense.yourdomain.de`
* Go to "Firewall > NAT > Port Forward"
* Create a new NAT rule
    * Interface:                Proxy VLAN
    * Destination:              ProxmoxGateway-Alias (10.10.10.0)
    * Destination Port from:    (other) 8006
    * Destination Port to:      (other) 8006
    * Redirect Target IP:       ProxmoxGateway-Alias (10.10.10.0)
    * Redirect Target Port:     (other) 8006
* Safe

<br>

### Add Proxy (proxmox.yourdomain.de)
* Open `proxy.yourdomain.de`
* Go to "Hosts > Proxy Hosts"
* Click green "Add Proxy Host"-Button
* Now enter the following values
    * Details-Tab
        * Domainname:               proxmox.yourdomain.de
        * Scheme:                   https
        * Forward Hostname / IP:    10.10.10.0 (Proxmox Gateway)
        * Forward Port:             8006
        * Websockets Support:       True
    * SSL-Tab
        * SSL Certificate:          "Requet a new SSL Certificalte"
        * Force SSL:                True
        * HTTP/2 Support:           True
        * Email Address for LE:     Your E-Mailadress
        * I Agree to the LE ToS:    True
* Click Safe-button

It will now take a while as the SSL certificate is created.<br>

Congratulations. You can now access your Proxmox via `proxmox.yourdomain.de`.<br>
We have now regulated all three important accesses via their own subdomain.<br>
Now we can continue with the next steps in a relaxed manner.
