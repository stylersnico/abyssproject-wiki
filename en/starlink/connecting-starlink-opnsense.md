---
title: Connecting Starlink to OPNSense
description: Connecting Starlink in bridge mode to a OPNSense firewall
published: true
date: 2022-09-24T13:33:40.763Z
tags: opnsense, starlink, bridge
editor: markdown
dateCreated: 2022-09-24T13:33:40.763Z
---

# Introduction
The goal of this procedure is to connect your Starlink router in bridge mode to your OPNSense firewall.

> You will need the Starlink RJ45 adapter if you don't already have it to put your router in bridge mode: https://shop.starlink.com/products/fr-consumer-ethernet-adapter-gen2
{.is-warning}

![starlink-ethernet.jpg](/starlink/starlink-ethernet.jpg)

# Configuring the bridge mode on the Starlink

Connect to the default Wi-Fi network and open the starlink application on your phone.
Now, go to **Settings** and enable **Bypass Starlink Wi-Fi router**:

![connecting-starlink-opnsense-01.png](/starlink/connecting-starlink-opnsense-01.png)

Wait 5 minutes for the starlink router to restart, then connect the ethernet cable to the **WAN** interface of your OPNSense firewall.

# Configuring OPNSense

Go to **Interfaces** -> **WAN** then configure the interface like this with **DHCPv4**.
> You must untick **Block private networks** or you will not get any IP address since starlink user carrier grade NAT network (100.64/10).
{.is-warning}


![connecting-starlink-opnsense-02.png](/starlink/connecting-starlink-opnsense-02.png)

Save interface settings.


Now, go to **System** -> **Gateways** -> **Single** and edit the default gateway of **WAN** interface:

![connecting-starlink-opnsense-03.png](/starlink/connecting-starlink-opnsense-03.png)

Put the IP address of a public DNS for example in monitoring in place of the default gateway provided by Starlink to have consistent monitoring (the default gateway will be always up because she is in local):

![connecting-starlink-opnsense-04.png](/starlink/connecting-starlink-opnsense-04.png)


You will finally be able to use your Starlink access:

![connecting-starlink-opnsense-05.png](/starlink/connecting-starlink-opnsense-05.png)