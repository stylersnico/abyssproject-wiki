---
title: Opening ports behind a Starlink connection 
description: Opening ports behind a Starlink connection with PureVPN
published: true
date: 2022-10-05T14:02:10.867Z
tags: opnsense, starlink, purevpn, nat
editor: markdown
dateCreated: 2022-09-24T13:56:37.351Z
---

> **PureVPN isn't that great**, it let this here to help you configure OpenVPN.
> If you wish to use your own VPS to do a VPN, here is a new guide: https://wiki.abyssproject.net/en/starlink/port-opening-behind-starlink-wireguard
{.is-danger}



# Introduction
The goal of this guide is to be able to open an inbound port on a Starlink connection by using an external VPN like PureVPN here.

> We will use PureVPN, but we don't have any affiliation with them.
> You must have the subscription with the **Port Forwarding** add-on:https://www.purevpn.com/port-forwarding
{.is-info}

You also need a firewall like OPNSense behind your Starlink connection that already act as a firewall and then, will allow us to manage the ports we need.

Example of OPNSense configuration with Starlink: https://wiki.abyssproject.net/en/starlink/connecting-starlink-opnsense


# Configuring PureVPN

Open your subscription at PureVPN and configure the **Port forwarding** add-on like this: https://my.purevpn.com/v2/dashboard/subscriptions

![port-opening-behind-starlink-purevpn-01.png](/starlink/nat-behind-starlink/port-opening-behind-starlink-purevpn-01.png)


Enable all ports, it only suitable is you have a firewall like us behind, if you don't have any firewall don't do this:

![port-opening-behind-starlink-purevpn-02.png](/starlink/nat-behind-starlink/port-opening-behind-starlink-purevpn-02.png)


# Configuring OPNSense firewall

## Configuring OpenVPN client

Download latest configuration files from here (**Others routers**): https://support.purevpn.com/openvpn-files.

Go to **System** -> **Trust** -> **Authorities** and click on **+**:

![port-opening-behind-starlink-purevpn-03.png](/starlink/nat-behind-starlink/port-opening-behind-starlink-purevpn-03.png)

Import the authority certificate from the archive (The content of **ca2.crt** file):

![port-opening-behind-starlink-purevpn-04.png](/starlink/nat-behind-starlink/port-opening-behind-starlink-purevpn-04.png)

Go to **VPN** -> **OpenVPN** -> **Clients** and click on **+**:
![port-opening-behind-starlink-purevpn-05.png](/starlink/nat-behind-starlink/port-opening-behind-starlink-purevpn-05.png)


> Your VPN username and password are available at the bottom of this page: https://my.purevpn.com/v2/dashboard/subscriptions
{.is-info}


You can also grab the server address and port in the VPN configuration file you want to use, like **fr2-ovpn-udp.ovpn**:

![port-opening-behind-starlink-purevpn-07.png](/starlink/nat-behind-starlink/port-opening-behind-starlink-purevpn-07.png)

Now, complete all the important information like this, you will need the content of **wdc.key** file for the **TLS Shared Key** :

![port-opening-behind-starlink-purevpn-06.png](/starlink/nat-behind-starlink/port-opening-behind-starlink-purevpn-06.png)

Go to **VPN** -> **OpenVPN** -> **Connection Status** and check that the connection is UP:

![port-opening-behind-starlink-purevpn-08.png](/starlink/nat-behind-starlink/port-opening-behind-starlink-purevpn-08.png)


## Configuring OpenVPN virtual interface

Go to **Intefaces** -> **Assignments** and assign the OpenVPN client virtual interface to OPT1 (or the first available one) :
![port-opening-behind-starlink-purevpn-09.png](/starlink/nat-behind-starlink/port-opening-behind-starlink-purevpn-09.png)


Configure only the description and enable the interface: 
![port-opening-behind-starlink-purevpn-10.png](/starlink/nat-behind-starlink/port-opening-behind-starlink-purevpn-10.png)


## Configuring routing

Go to **Firewall** -> **Aliases** and create the Alias containing all the hosts that should go on internet using the new VPN:

![port-opening-behind-starlink-purevpn-11.png](/starlink/nat-behind-starlink/port-opening-behind-starlink-purevpn-11.png)

Now, go to **Firewall** -> **LAN** and create the two following rules:

- The first rule allows the hosts in the **PureVPNClients** alias to go in and out via the VPN tunnel.
- The next rule allows everyone else to speak on the Starlink **WAN** directly without using the VPN tunnel.

![port-opening-behind-starlink-purevpn-12.png](/starlink/nat-behind-starlink/port-opening-behind-starlink-purevpn-12.png)

## Configuring outbound NAT

Go to **Firewall** -> **Outbound** and enable **Hybrid outbound NAT rule generation**.

Next, create those 3 rules (on the PureVPN interface) so the outside traffic is functional:

![port-opening-behind-starlink-purevpn-13.png](/starlink/nat-behind-starlink/port-opening-behind-starlink-purevpn-13.png)


## Port opening

First, do your port forward by going to **Firewall** -> **NAT** -> **Port Forward**.
You should use **PureVPN** interface and not **WAN** like this: 
![port-opening-behind-starlink-purevpn-14.png](/starlink/nat-behind-starlink/port-opening-behind-starlink-purevpn-14.png)

Now, create your inbound firewall rules from **Firewall** -> **Rules** -> **PureVPN** like this, so the port forward is allowed: 

![port-opening-behind-starlink-purevpn-15.png](/starlink/nat-behind-starlink/port-opening-behind-starlink-purevpn-15.png)


Then, you will be able to check that your port is well open with tools like this https://www.yougetsignal.com/tools/open-ports/ :

![port-opening-behind-starlink-purevpn-16.png](/starlink/nat-behind-starlink/port-opening-behind-starlink-purevpn-16.png)

## Sources
- M4DM4NZ : https://forum.opnsense.org/index.php?topic=4979.0