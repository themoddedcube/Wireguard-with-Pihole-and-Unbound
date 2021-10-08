# Full Tunnel or Split Tunnel IPv6 + IPv4 Wireguard VPN connections to an ad blocking Pi-Hole server, from your Android, iOS, Chrome OS, Linux, macOS, & Windows devices

<img src="./images/data-privacy-risk.svg" width="125" align="right">


The goal of this project is to enable you to safely and privately use the Internet on your phones, tablets, and computers with a self-run VPN Server in the cloud, or on your own hardware in your home. This software shields you from intrusive advertisements. It blocks your ISP, cell phone company, public WiFi hotspot provider, and apps/websites from gaining insight into your usage activity.

Both Full Tunnel (all traffic) and Split Tunnel (DNS traffic only) VPN connections provide DNS based ad-blocking over an encrypted connection to the device this is running on. The differences are:

- A Split Tunnel VPN allows you to interact with devices on your Local Network (such as a Chromecast or Roku).
- A Full Tunnel VPN can help bypass misconfigured proxies on corporate WiFi networks, and protects you from Man-In-The-Middle SSL proxies.

| Tunnel Type | Data Usage | Server CPU Load | Security | Ad Blocking |
| -- | -- | -- | -- | -- |
| full | +10% overhead for vpn | low | 100% encryption | yes
| split | just kilobytes per day | very low | dns encryption only | yes

While Pi-hole was originally authored to run on a Raspberry Pi, people have followed this guide to deploy securely hosted instances of Pi-hole with VPN only access on Google Cloud, AWS, Heroku, Azure, Linode, Digital Ocean, Oracle Cloud, and on spare hardware at home.

---

## Quickstart (Do not do steps 7 & 8 without doing the Client Setup Guide first)

1.  Download and execute **setup.sh** from this repository to:

    1.  install the latest Wireguard packages

    2.  install the latest Pi-Hole, and configure it to accept DNS requests from the Wireguard interface

    3.  display a QR Code for 1 Split Tunnel VPN Profile, so you can import the VPN Profile to your device without having to type anything

```bash
sudo su -
curl -O https://raw.githubusercontent.com/themoddedcube/Wireguard-with-Pihole-and-Unbound/master/setup.sh
chmod +x setup.sh
bash ./setup.sh 
```

2.  Make sure your router or firewall is forwarding incoming UDP packets on Port 51515 to the Server that you ran the **setup.sh** script on. You can actually use any port, just change "Server's WireGuard port" on setup to whatver port you want to use.

3.  Change the default pihole port by running `sudo nano /etc/lighttpd/lighttpd.conf` and change `server.port` to whatever port you want and save the script.

4.  After saving the script run `sudo service lighttpd restart` and connect to your pihole instance that you just created.

5. [Install and configure Unbound DNS server](https://docs.pi-hole.net/guides/dns/unbound/)

6. Check to see if packet forwarding is enabled for Wireguard by running `sudo nano /etc/sysctl.conf`, look for `net.ipv4.ip_forward=1` make sure it is uncommented, save the script and reboot the system.

7.  Create another VPN Client Profile by running `./setup.sh` again, you can create 253 profiles without modifying the script.

8. Open the Wireguard Application on your Client Device, and edit the VPN Profile.

   1.  Change the **Allowed IPs** to include your LAN subnet. For example, if your router's IP address is `192.168.86.1`, and your Wireguard server has an IP somewhere in the range of `192.168.86.2` to `192.168.86.255`, your subnet is `192.168.86.0/24`. If you add `192.168.86.0/24` to the comma separated list of **Allowed IPs** in the Client Configuration file, you will be able to ping any device with an IP address between `192.168.86.1` to `192.168.86.254` over your Wireguard connection. If you wish to route all your traffic through the VPN (Full Tunnel), edit the **Allowed IPs** on your Client Profile on your device to read `0.0.0.0/0, ::/0`.


---

## Client Setup Guide

To connect and use the VPN, you will need to install the Wireguard VPN software on your device or computer, information on how to install this software can be found on google.

## Delete Clients from Server

Print list of all clients on the server:

```bash
sudo wg show
```

Sample output may look like this:

> ```
> peer: txUZ0iqCyu69qQFq08U420hOp3/A4lYtrHVrJrAYBys=
>   preshared key: (hidden)
>   endpoint: 99.99.99.99:99999
>   allowed ips: 10.66.66.2/32, fd42:42:42::2/128
>   latest handshake: 4 days, 20 hours, 4 minutes, 20 seconds ago
>   transfer: 4.20 MiB received, 4.20 MiB sent
> ```

Make note of the unique string after the word **peer:** for the client you wish to delete. In the example above, it is `txUZ0iqCyu69qQFq08U420hOp3/A4lYtrHVrJrAYBys=`.

Remove the client:

```bash
sudo wg set wg0 peer txUZ0iqCyu69qQFq08U420hOp3/A4lYtrHVrJrAYBys= remove
```

Replace `txUZ0iqCyu69qQFq08U420hOp3/A4lYtrHVrJrAYBys=` in the command above with the appropriate **peer:** you wish to delete on your server.\

---

## Edge Case Requirements

### Configure automated Pi-Hole updates and scheduled reboots

Pause and consider if you need this for mission critical Pi-hole Servers. If you are running multiple Pi-Holes for redundancy, and you choose to implement this, stagger the upgrade and reboot schedules. Be prepared to perform health-checks to ensure all services are operational. Blind upgrades are not gauranteed to be smooth.

**Note:** The following steps assume you have **nano** installed. You can use any other editor (e.g **vim**) to do this.

Create the script to check if a reboot is required or not, by checking for the presence of the **/var/run/reboot-required** file, by running:

```bash
sudo nano /etc/cron.daily/zz-restart-if-required
```

Paste the following into **/etc/cron.daily/zz-restart-if-required**:

> ```bash
> #!/bin/sh
> if [ -f /var/run/reboot-required ]; then
>   /sbin/shutdown -r now
> fi
> ```

Set the correct permissions:

```bash
sudo chmod 755 /etc/cron.daily/zz-restart-if-required
```

Check for Pi-Hole updates and perform an update if one is available:

Create the script to update PiHole:

```bash
sudo nano /etc/cron.daily/update-pi-hole
```

Paste the following into **/etc/cron.daily/update-pi-hole**:

> ```bash
> #!/bin/sh
> /usr/local/bin/pihole -up
> ```

Set the correct permissions:

```bash
sudo chmod 755 /etc/cron.daily/update-pi-hole
```

### Enabling or Blocking communication between Wireguard Clients

If you wish to enable communication between select Wireguard clients, using the same CIDR notation under **Allowed IPs** in each Client Configuration file is necessary. This table could help you plan which devices get what IPs.

**TODO:** provide a subnet cheatsheet for IPv6 addresses

#### Subnet Cheatsheet

| CIDR Notation | Address Range |
| -- | -- |
| 10.66.66.0/30 | 10.66.66.1 - 10.66.66.2 |
| 10.66.66.0/29 | 10.66.66.1 - 10.66.66.6 |
| 10.66.66.0/28 | 10.66.66.1 - 10.66.66.14 |
| 10.66.66.0/27 | 10.66.66.1 - 10.66.66.30 |
| 10.66.66.0/26 | 10.66.66.1 - 10.66.66.62 |
| 10.66.66.0/25 | 10.66.66.1 - 10.66.66.126 |
| 10.66.66.0/24 | 10.66.66.1 - 10.66.66.254 |

## Contributions Welcome

If there is something that can be done better, or if this documentation can be improved in any way, please submit a Pull Request with your fixes or edits.

---



