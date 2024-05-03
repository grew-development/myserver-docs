# Tailscale Installation
In Progress
Table of contents

1. <a href="#1-why-did-i-use-tailscale">Why did I use Tailscale?</a>
2. <a href="#2-create-a-account">Create a Account</a>
3. <a href="#3-download-and-install-tailscale-on-client">Download and Install Tailscale on Client</a>
4. <a href="#4-download-and-install-tailscale-on-opnsense">Download and Install Tailscale on OPNsense</a>
    * <a href="#updating-tailscale-in-opnsense">Updating Tailscale in OPNsense</a>
5. <a href="#5-edit-rfc1918-rule">5. Edit RFC1918-Rule</a>
6. <a href="#6-allowing-access-from-tailscale-to-proxy">6. Allowing Access from tailscale to proxy</a>
7. <a href="#7-allowing-access-from-tailscale-to-opnsense">7. Allowing Access from tailscale to opnsense</a>
8. <a href="#8-change-dns-record">8. Change DNS-Record</a>
9. <a href="#9-add-a-tailscale-access-list">9. Add a Tailscale Access List</a>

<br><br>

## 1. Why did I use Tailscale?

* **Better security**: With Tailscale, the separation of public and private services on my server is ensured, so that,<br>
for example, my firewall or other desired services are only accessible to me, i.e. internally.

* **SSh and Filesharing**: Tailscale allows me to share files or other things remotely via SSH and or SMTP.<br>
Which simplifies server management remotely, so you don't always have to be at home

* **Personal VPN**: Tailscale offers the ability to turn my server into my own VPN,<br>
providing an additional layer of privacy and security for my online activities.

* **Cost-effective Solution**: Tailscale is free to use, making it an economical choice for securing and managing my server infrastructure.

Again, [@Redacks](https://github.com/redacks) recommended me to use this for all these reasons. As you have probably already noticed that I enjoy working with Redacks often, I also trust his knowledge and his own experiences in this area. And I can link not only my server, but many other accounts and programs with Tailscale.

<br><br>

## 2. Create a Account
Go to [https://tailscale.com/](https://tailscale.com/) and Create a Account.
I use github access to register, but you can use whatever you like.

<br><br>

## 3. Download and Install Tailscale on Client
Then click on the download button above to download the current version for your operating system.
Download and run the Installer. Click on Log in from the Tailscale icon now in your system tray and authenticate in your browser and<br>
sign up with your provider do you like.

Here you will also find the direct link to the different operating systems:
* [Windows - click here](https://pkgs.tailscale.com/stable/tailscale-setup-latest.exe)
* [macOS - click here](https://apps.apple.com/ca/app/tailscale/id1475387142?mt=12)
* [Linux]() - `curl -fsSL https://tailscale.com/install.sh | sh`

<br><br>

## 4. Download and Install Tailscale on OPNsense
After you have successfully registered, we can now start installing Tailscale on our servers.<br>
I'll just start by installing Tailscale on OPNsense, the reason for this is that nobody except me can access my firewall privately to delete,<br>
edit or anything else. 

The great thing is that tailscale itself has written a documentation on how to install Tailscale in the OPNsense.
Since this is a complete installation guide, I will only link it here.

[Link to the documentary](https://tailscale.com/kb/1097/install-opnsense)

>[!NOTE]
> After installation and after connection,<br>
> Tailscale writes that in the list of interfaces in the OPNsense interface,<br>
> the Tailscale network should appear.
>
> If this is not the case, you should add the interface under `Interfaces > Assignments > new interface`.

<br>

### Updating Tailscale in OPNsense
[Link to the documentary](https://tailscale-com.translate.goog/kb/1097/install-opnsense?_x_tr_sl=en&_x_tr_tl=de&_x_tr_hl=de&_x_tr_pto=wapp&_x_tr_hist=true#updating-tailscale)

<br><br>

## 5. Edit RFC1918-Rule
To update the private IP addresses and their aliases with the new interface, I would do this next. To do this, proceed as follows

* Go to `Firewall > Aliases`
* Click "edit" on the RFC1918-Rule
* Add the new network (in my example __opt4_network) in the Content area
* Safe and Apply

> [!NOTE]
> I hope you've added all new interfaces to this rule so far, if not... do it now!

<br><br>

## 6. Allowing Access from tailscale to proxy
* Open `opnsense.yourdomain.de`
* Go to "Firewall > NAT > Port Forward"
* Create a new NAT rule
    * Interface:                Tailscale
    * Destination:              Tailscale_ADDRESS
    * Destination Port from:    http (80)
    * Destination Port to:      http (80)
    * Redirect Target IP:       Proxy-Alias (10.1.2.2)
    * Redirect Target Port:     http (80)
* Safe

Repeat creating a NAT rule only with the ports HTTPS (443)

<br><br>

## 7. Allowing Access from tailscale to opnsense
* Open `opnsense.yourdomain.de`
* Go to "Firewall > NAT > Port Forward"
* Create a new NAT rule
    * Interface:                Tailscale
    * Destination:              Tailscale_ADDRESS
    * Destination Port from:    (other) 9443 - Your Opnsense WebPort
    * Destination Port to:      (other) 9443 - Your Opnsense WebPort
    * Redirect Target IP:       127.0.0.1
    * Redirect Target Port:     (other) 9443 - Your Opnsense WebPort
* Safe

We took this step for security reasons, so that even if the proxy is turned off, we can still access our OPNsense.
Believe me, I managed to do that once during the installation xD

<br><br>

## 8. Change DNS-Record
Now we have to change our DNS record with our provider to the tailscale address.<br>
After you have successfully authenticated yourself with your browser when connecting to tailscale, you should see your opnsense at `https://login.tailscale.com/admin/machines`.

In the “Addresses” column you can now copy this IP and change it in your DNS entries.
It's best just for `proxmox.yourdomain.de` at first. You can test this after a while by pinging this subdomain `ping proxmox.yourdomain.de`.<br>
As the answer IP, your Tailscale address should now appear.

If this worked, you can change the DNS values of all important domains. Among other things, from
* opnsense.yourdomain.de
* proxy.yourdomain.de

Important to know: you should change all IP addresses that should not be accessed from outside.

<br><br>

## 9. Add a Tailscale Access List
We can prevent connections to the outside world with a DNS rebind attack. Our NGINX-Proxy-Manager also helps us to do this.

* Go to `proxy.yourdomain.de`
* Switch to tab Access List and Added a new List
* Give the list a new name, I call this list "Tailsclale"
* Go to Access-Tab and enter the Tailscale subnet there.
    * Subnet: 100.64.0.0/10
* Safe

<br>

* Now Switch to your Host `Host > Proxy Hosts`
* Edit the Source `proxmox.yourdomain.de`
* Details-Tab
    * Access List: YOUR_LIST
* Safe

Now only you can access proxmox.yourdomain.de via your Tailscale IP connection.<br>
As soon as your DNS entry has been updated worldwide, you will be directed to this domain.<br>
