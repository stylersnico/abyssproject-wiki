---
title: Gateway IPv4 + IPv6 with Netplan
description: Get default gateway with both IPv4 and IPv6 via Netplan
published: true
date: 2023-07-11T14:20:28.767Z
tags: netplan, ubuntu, ipv6, ipv4
editor: markdown
dateCreated: 2023-07-11T14:20:28.767Z
---

# Introduction

The goal of this guide is to have a default gateway for both IPv4 and IPv6 at the same time via netplan interface configuration.


# Configuring the gateways

Open your interface config file, here for example:
```bash
nano /etc/netplan/00-installer-config.yaml
```
 
Add your route like this: 
```bash
      routes:
      - to: default
        via: 192.168.100.1
      - to: "::/0"
        via: fde8:b429:841e:b651::1
```

Here is my complete configuration, for example: 

```bash
network:
  ethernets:
    ens18:
      addresses:
      - 192.168.100.100/24
      - fde8:b429:841e:b651::100/64
      nameservers:
        addresses:
        - 1.1.1.1
        - 8.8.8.8
        - 2606:4700:4700::1111
        - 2606:4700:4700::1001
        search: []
      routes:
      - to: default
        via: 192.168.100.1
      - to: "::/0"
        via: fde8:b429:841e:b651::1
  version: 2
```

Now, test your configuration with this command: 

```bash
netplan try -timeout 30
```

If the configuration is OK, apply it or restart the server: 

```bash
netplan apply
```