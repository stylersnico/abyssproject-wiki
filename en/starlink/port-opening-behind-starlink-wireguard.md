---
title: Opening ports behind a Starlink connection with a WireGuard tunnel
description: Opening ports behind a Starlink connection with a WireGuard tunnel
published: true
date: 2022-10-05T14:13:01.496Z
tags: opnsense, starlink, nat, wireguard
editor: markdown
dateCreated: 2022-10-05T14:00:33.476Z
---

# Introduction
The goal of this guide is to port forward port on a Starlink connection with an external Wireguard server.

> Is this guide, we will use a OVH vps, I have no affiliation with them.
{.is-info}

You also need a firewall like OPNSense on your Starlink connection that already act as a firewall and after will allow us to open the wanted ports.


Configuration example with OPNSense: https://wiki.abyssproject.net/en/starlink/connecting-starlink-opnsense


# Configuring Wireguard server

In this guide, we will use a vps starter from OVH: https://www.ovh.com/fr/order/vps/?v=3#/vps/

Install Wireguard with the Nyr's script available here: https://github.com/Nyr/wireguard-install 

```bash
wget https://git.io/wireguard -O wireguard-install.sh && bash wireguard-install.sh
```

At the end, the .conf file will be created with all mandatory information:
```bash
root@vps:~# cat opnsense.conf
[Interface]
Address = 10.7.0.2/24
DNS = 1.1.1.1, 1.0.0.1
PrivateKey = XXX
[Peer]
PublicKey = XXX
PresharedKey = XXX
AllowedIPs = 0.0.0.0/0, ::/0
Endpoint = XXX:51820
PersistentKeepalive = 25
root@vps:~#
```

# Configuring OPNSense firewall

## Configuration of the WireGuard client

Go to **System** -> **Firmware** -> **Plugins** and install **os-wireguard** plugin like this:

![opnsense-wireguard-nat-01.png](/starlink/nat-behind-starlink/wireguard/opnsense-wireguard-nat-01.png)


Go to **VPN** -> **WireGuard** -> **Endpoint** and add an endpoint: 

![opnsense-wireguard-nat-02.png](/starlink/nat-behind-starlink/wireguard/opnsense-wireguard-nat-02.png)

Add the endpoint like this with the information of the **[Peer]** section of the config file:

![opnsense-wireguard-nat-03.png](/starlink/nat-behind-starlink/wireguard/opnsense-wireguard-nat-03.png)

Go to **Local** and click on **+**:

![opnsense-wireguard-nat-04.png](/starlink/nat-behind-starlink/wireguard/opnsense-wireguard-nat-04.png)

Add the local interfaces with the information of the **[Interface]** section of the configuration file. You need to disable the routes:

![opnsense-wireguard-nat-05.png](/starlink/nat-behind-starlink/wireguard/opnsense-wireguard-nat-05.png)

Enable the Wireguard service from the **General** tab:
![opnsense-wireguard-nat-06.png](/starlink/nat-behind-starlink/wireguard/opnsense-wireguard-nat-06.png)

Wait some time and go to **List Configuration** to check the communication:
![opnsense-wireguard-nat-07.png](/starlink/nat-behind-starlink/wireguard/opnsense-wireguard-nat-07.png)


## Configuring Wireguard interface

Go to **Interfaces** -> **Assignments** and assign **wg1** on the first available optional port: 

![opnsense-wireguard-nat-08.png](/starlink/nat-behind-starlink/wireguard/opnsense-wireguard-nat-08.png)

Go to **Interfaces** -> **Wireguard/opt1** -> Enable the interface without any IP configuration:

![opnsense-wireguard-nat-09.png](/starlink/nat-behind-starlink/wireguard/opnsense-wireguard-nat-09.png)


## Configuring the gateway

Go to **System** -> **Gateways** -> **Single** and add the gateway:

![opnsense-wireguard-nat-10.png](/starlink/nat-behind-starlink/wireguard/opnsense-wireguard-nat-10.png)


Add the gateway like this, you need to use the **Wireguard** interface created before, and you have to tick  **Far Gateway**: 

![opnsense-wireguard-nat-11.png](/starlink/nat-behind-starlink/wireguard/opnsense-wireguard-nat-11.png)


## Configuring the firewall rules

To allow communication in the tunnel, you need to create the following rule: 

![opnsense-wireguard-nat-12.png](/starlink/nat-behind-starlink/wireguard/opnsense-wireguard-nat-12.png)


## Configuring outbound client communication

Create the alias WireguardClients that will contain all the hosts that should go on Internet with the Wireguard tunnel:

![opnsense-wireguard-nat-13.png](/starlink/nat-behind-starlink/wireguard/opnsense-wireguard-nat-13.png)

Now, create the two following rules, the first one allow clients in the Wireguard alias to go on internet with the tunnel, the next one allow everyone else to go outside via the WAN:

![opnsense-wireguard-nat-14.png](/starlink/nat-behind-starlink/wireguard/opnsense-wireguard-nat-14.png)


## Configuring Outbound Nat

Go to **Firewall** -> **NAT** -> **Outbound** and put the firewall in **Hybrid outbound NAT rule Generation**.

Then, create the following rules: 

![opnsense-wireguard-nat-15.png](/starlink/nat-behind-starlink/wireguard/opnsense-wireguard-nat-15.png)


## Port forwarding

Go to **Firewall** -> **NAT** -> **Port Forward** et create the rules you need like this:
![opnsense-wireguard-nat-16.png](/starlink/nat-behind-starlink/wireguard/opnsense-wireguard-nat-16.png)

Go to **Firewall** -> **Rules** -> **Wireguard**  and create the rules according to the port forward you did before:
![opnsense-wireguard-nat-17.png](/starlink/nat-behind-starlink/wireguard/opnsense-wireguard-nat-17.png)


# Forwarding port on the VPS
Now we are back on the VPS, we don't need to do anything else on OPNSense.

## Port forwarding with IPTables
Enable port forwarding on the system with this command:
```bash
sysctl -w net.ipv4.ip_forward=1
```

Enable it permanently by adding this line in the file `/etc/sysctl.conf`: 
```bash
net.ipv4.ip_forward = 1
```

Redirect the port that you want with the following command (adapt to the port you want to open):

```bash
iptables -t nat -A PREROUTING -p tcp --dport 443 -j DNAT --to-destination 10.7.0.2:443
iptables -t nat -A PREROUTING -p tcp --dport 80 -j DNAT --to-destination 10.7.0.2:80
iptables -t nat -A PREROUTING -p udp --dport 1194 -j DNAT --to-destination 10.7.0.2:1194
iptables -t nat -A PREROUTING -p udp --dport 514 -j DNAT --to-destination 10.7.0.2:1194
iptables -t nat -A POSTROUTING -j MASQUERADE
```

## Saving IPTables config

Install the following package:
```bash
apt-get install iptables-persistent
```

Backup the rules with this command: 
```bash
iptables-save > /etc/iptables/rules.v4
```

## Test

At the end, you will be able to check that the port is open with tools like https://www.yougetsignal.com/tools/open-ports/:

![port-opening-behind-starlink-purevpn-16.png](/starlink/nat-behind-starlink/port-opening-behind-starlink-purevpn-16.png)


## Sources

- https://github.com/Nyr/wireguard-install
- https://docs.opnsense.org/manual/how-tos/wireguard-selective-routing.html