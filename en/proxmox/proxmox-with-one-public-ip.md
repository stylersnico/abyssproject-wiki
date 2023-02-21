---
title: Using Proxmox with one public IP address
description: Using Proxmox at Kimsufi, Hetzner, or others with only one IP, NATfor VMs and IPv6
published: true
date: 2023-02-21T08:24:17.715Z
tags: debian, hetzner, proxmox, kimsufi
editor: markdown
dateCreated: 2023-02-20T17:23:47.585Z
---

# Introduction
The goal of this guide is to set up a Proxmox at a hoster who gives you only one public IP (V4 and / or V6) on a dedicated server.
The goal is that all virtual machines have Internet and that you can forward port to them.

> This guide is clearly not a best practice.
> However, all my services are hosted on this, it's working and for personnal use it's just fine.
{.is-warning}


# Prerequisite

Your server should already have the latest Proxmox release on it and the IP address should be static like this:

![proxmox-with-one-public-ip-00.png](/proxmox/proxmox-with-one-public-ip/proxmox-with-one-public-ip-00.png)


# How it's working

The external network (VMBR0) has a public IPv4 and a public IPv6 available:
- `IPv4 network: 37.187.147.215/24`
- `IPv6 network: 2001:41d0:a:53d7::1/128`

Before starting, we need to create a private network.

In this guide, the internal network (VMBR1) will have IPv4 and IPv6:
- `IPv4 network: 192.168.100.1/24`
- `IPv6 network: fde8:b429:841e:b651::1/64`

> The IPv6 range is arbitrary and generated with this tool: https://simpledns.plus/private-ipv6.
He is valid for our usage.
{.is-info}

The communication will behave like this: **VMBR0 <-> VMBR1 <-> Virtual machines**.

The virtual machines will have address on the internal network (VMBR1), here is an example for a virtual machine:
```bash
iface ens18 inet static
        address 192.168.100.104/24
        gateway 192.168.100.1
        dns-nameservers 1.1.1.1 1.0.0.1


iface ens18 inet6 static
        address fde8:b429:841e:b651::104/64
        gateway fde8:b429:841e:b651::1
        dns-nameservers 2606:4700:4700::1111 2606:4700:4700::1001
```

The Proxmox will manage the NAT.


# Configuring UFW

To protect Proxmox, we will install UFW (this is not mandatory, but do it anyway).

Install UFW: 
```bash
apt-get install ufw -y
```

Open the default config file: 
```bash
nano /etc/default/ufw
```

Configure the following options: 
```bash
IPV6=yes
DEFAULT_FORWARD_POLICY="ACCEPT"
```

Then open this configuration file: 
```bash
nano /etc/ufw/sysctl.conf
```

And configure the following lines: 
```bash
net/ipv4/ip_forward=1
net/ipv6/conf/default/forwarding=1
net/ipv6/conf/all/forwarding=1
```

Disable the firewall:
```bash
ufw disable
```

Allow all outside connections and deny all incoming connections:
```bash
ufw default deny incoming
ufw default allow outgoing
```

If you have a fixed IP address at home, you can allow all connections from it:
```bash
ufw allow from 45.23.28.24
```

If you don't have any, allow all connections to SSH and Proxmox: 
```bash
ufw allow 22
ufw allow 8006
```

Finally, enable the firewall: 
```bash
ufw enable
```

> If you want to use a dynamic IP with UFW (in French): https://www.abyssproject.net/2017/07/utiliser-ip-dynamique-nginx-ufw/
{.is-info}



# Internal Network setup (VMBR1)

Open the Proxmox web interface and create the new interface from here (**System** -> **Network** -> **Create** -> **Linux Bridge**):

![proxmox-with-one-public-ip-01.png](/proxmox/proxmox-with-one-public-ip/proxmox-with-one-public-ip-01.png)

Configure the interface like this with the private networks we've seen before: 

![proxmox-with-one-public-ip-02.png](/proxmox/proxmox-with-one-public-ip/proxmox-with-one-public-ip-02.png)

> You will seen that I don't put a **Bridge port**.
> If you put a virtual machine or virtual switch on **VMBR0** without additional IP, it is possible than your network port will be shutdown at the datacenter.
{.is-danger}


# Routing configuration

We will configure **MASQUERADE** under Linux.
To be easy, **MASQUERADE** is a **1-to-many NAT type**.

Behind that rude explanation is hidden the most common NAT type that you have behind your provider router and behind any firewall.
All the computers from the internal network can go to the Internet with only one public IP address.

Open your interfaces' configuration:
```bash
nano /etc/network/interfaces
```

Add the following lines to **VMBR1** in **IPv4**:
```bash
post-up echo 1 > /proc/sys/net/ipv4/ip_forward
post-up iptables -t nat -A POSTROUTING -s 192.168.100.0/24 -o vmbr0 -j MASQUERADE
post-down iptables -t nat -D POSTROUTING -s 192.168.100.0/24 -o vmbr0 -j MASQUERADE
```

Add the following lines to **VMBR1** in **IPv6**: 
```bash
post-up echo 1 > /proc/sys/net/ipv6/conf/all/forwarding
post-up ip6tables -t nat -A POSTROUTING -s fde8:b429:841e:b651::1/64 -o vmbr0 -j MASQUERADE
post-down ip6tables -t nat -D POSTROUTING -s fde8:b429:841e:b651::1/64 -o vmbr0 -j MASQUERADE
```

In a meaningful way, your full interface config should look just like this: 

```bash
auto lo
iface lo inet loopback

iface eno1 inet manual

iface eno2 inet manual

auto vmbr0
iface vmbr0 inet static
        address 37.187.147.215/24
        gateway 37.187.147.254
        bridge-ports eno1
        bridge-stp off
        bridge-fd 0
        hwaddress 0C:C4:7A:47:DA:4C

iface vmbr0 inet6 static
        address 2001:41d0:a:53d7::1/128
        gateway 2001:41d0:000a:53ff:00ff:00ff:00ff:00ff

auto vmbr1
iface vmbr1 inet static
        address 192.168.100.1/24
        bridge-ports none
        bridge-stp off
        bridge-fd 0
        post-up echo 1 > /proc/sys/net/ipv4/ip_forward
        post-up iptables -t nat -A POSTROUTING -s 192.168.100.0/24 -o vmbr0 -j MASQUERADE
        post-up /root/dnat.sh
        post-down iptables -t nat -D POSTROUTING -s 192.168.100.0/24 -o vmbr0 -j MASQUERADE

iface vmbr1 inet6 static
        address fde8:b429:841e:b651::1/64
        post-up echo 1 > /proc/sys/net/ipv6/conf/all/forwarding
        post-up ip6tables -t nat -A POSTROUTING -s fde8:b429:841e:b651::1/64 -o vmbr0 -j MASQUERADE
        post-down ip6tables -t nat -D POSTROUTING -s fde8:b429:841e:b651::1/64 -o vmbr0 -j MASQUERADE
```

From here, **restart** your Proxmox and your virtual machines will have internet if you put fixed IP address on their network cards.


# DHCP for virtual machines

To ease your life and virtual machines installation, you can set up a small DHCPv4 server that will give a pool of adress to your virtual machines.

Install DNSMasq: 

```bash
apt install dnsmasq -y
```

Open the config file:
```bash
nano /etc/dnsmasq.conf
```

Configure a range for VMBR1 like this:

```bash
# interface to listen to
interface=vmbr1
# IP range to handout
dhcp-range=192.168.100.200,192.168.100.210,30d
# set gateway
dhcp-option=vmbr1,3,192.168.100.1
# DNS server
server=1.1.1.1
server=8.8.8.8
# DHCP database location (to save to)
dhcp-leasefile=/var/lib/misc/dnsmasq.leases
```

Enable and start the service: 
```bash
systemctl enable dnsmasq && systemctl start dnsmasq
```

# Fixed IP setup for virtual machines

Here, we will take the case of a virtual machine under Debian 11 with the virtual network card on the **VMBR1** virtual switch:

```bash
# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
allow-hotplug ens18
iface ens18 inet static
        address 192.168.100.104/24
        gateway 192.168.100.1
        dns-nameservers 1.1.1.1 1.0.0.1


iface ens18 inet6 static
        address fde8:b429:841e:b651::104/64
        gateway fde8:b429:841e:b651::1
        dns-nameservers 2606:4700:4700::1111 2606:4700:4700::1001
```

# Port forwarding to virtual machines


Here we will create a script to open ports needed on the VMs.

Here, the port 80 will be open in both IPv4 and IPv6 by redirecting to the IP **.104**.

Create this script for your configuration:

```bash
nano /root/dnat.sh
```

Here, open the port 80 on the virtual machine **.104** in both IPv4 and IPv6: 

```bash
sleep 60
iptables -t nat -A PREROUTING -i vmbr0 -p tcp --dport 80 -j DNAT --to-destination 192.168.100.104:80
ip6tables -t nat -A PREROUTING -i vmbr0 -p tcp -m tcp --dport 80 -j DNAT --to-destination [fde8:b429:841e:b651::104]:80
```

> **iptables** manage NAT rules in **IPv4**. **ip6tables** manage NAT rules in **IPv6**.
{.is-info}

Beware, if you configured UFW you also need to allow the port in the firewall:

```bash
ufw allow 80
```

Then, you need to set up the script in the interfaces' config file: 

```bash
nano /etc/network/interfaces
```

Add the following lines to VMBR1 in IPv4:

```bash
post-up echo 1 > /proc/sys/net/ipv4/ip_forward
post-up iptables -t nat -A POSTROUTING -s 192.168.100.0/24 -o vmbr0 -j MASQUERADE
post-up /root/dnat.sh
post-down iptables -t nat -D POSTROUTING -s 192.168.100.0/24 -o vmbr0 -j MASQUERADE
```

Like this, the NAT will be applied at each restart.

Here is the file that I personally use: 

```bash
sleep 60

#SSH
iptables -t nat -A PREROUTING -i vmbr0 -p tcp --dport 2201 -j DNAT --to-destination 192.168.100.101:22
iptables -t nat -A PREROUTING -i vmbr0 -p tcp --dport 2202 -j DNAT --to-destination 192.168.100.102:22
iptables -t nat -A PREROUTING -i vmbr0 -p tcp --dport 2203 -j DNAT --to-destination 192.168.100.103:22
iptables -t nat -A PREROUTING -i vmbr0 -p tcp --dport 2204 -j DNAT --to-destination 192.168.100.104:22
iptables -t nat -A PREROUTING -i vmbr0 -p tcp --dport 2205 -j DNAT --to-destination 192.168.100.105:22
iptables -t nat -A PREROUTING -i vmbr0 -p tcp --dport 2206 -j DNAT --to-destination 192.168.100.106:22

#SNMP
iptables -t nat -A PREROUTING -i vmbr0 -p udp --dport 2161 -j DNAT --to-destination 192.168.100.101:161
iptables -t nat -A PREROUTING -i vmbr0 -p udp --dport 2162 -j DNAT --to-destination 192.168.100.102:161
iptables -t nat -A PREROUTING -i vmbr0 -p udp --dport 2163 -j DNAT --to-destination 192.168.100.103:161
iptables -t nat -A PREROUTING -i vmbr0 -p udp --dport 2164 -j DNAT --to-destination 192.168.100.104:161
iptables -t nat -A PREROUTING -i vmbr0 -p udp --dport 2165 -j DNAT --to-destination 192.168.100.105:161

#Swizzin
iptables -t nat -A PREROUTING -i vmbr0 -p tcp --dport 7443 -j DNAT --to-destination 192.168.100.106:7443

#reverse proxy
iptables -t nat -A PREROUTING -i vmbr0 -p tcp --dport 80 -j DNAT --to-destination 192.168.100.104:80
iptables -t nat -A PREROUTING -i vmbr0 -p tcp --dport 443 -j DNAT --to-destination 192.168.100.104:443
ip6tables -t nat -A PREROUTING -i vmbr0 -p tcp -m tcp --dport 80 -j DNAT --to-destination [fde8:b429:841e:b651::104]:80
ip6tables -t nat -A PREROUTING -i vmbr0 -p tcp -m tcp --dport 443 -j DNAT --to-destination [fde8:b429:841e:b651::104]:443

#VPN
iptables -t nat -A PREROUTING -i vmbr0 -p udp --dport 2194 -j DNAT --to-destination 192.168.100.103:2194
ip6tables -t nat -A PREROUTING -i vmbr0 -p udp -m udp --dport 2194 -j DNAT --to-destination [fde8:b429:841e:b651::103]:2194

#Bastillion
iptables -t nat -A PREROUTING -i vmbr0 -p tcp --dport 8443 -j DNAT --to-destination 192.168.100.101:8443
ip6tables -t nat -A PREROUTING -i vmbr0 -p tcp -m tcp --dport 8443 -j DNAT --to-destination [fde8:b429:841e:b651::101]:8443
```

The complete interface file looks like this: 

```bash
auto lo
iface lo inet loopback

iface eno1 inet manual

iface eno2 inet manual

auto vmbr0
iface vmbr0 inet static
        address 37.187.147.215/24
        gateway 37.187.147.254
        bridge-ports eno1
        bridge-stp off
        bridge-fd 0
        hwaddress 0C:C4:7A:47:DA:4C

iface vmbr0 inet6 static
        address 2001:41d0:a:53d7::1/128
        gateway 2001:41d0:000a:53ff:00ff:00ff:00ff:00ff

auto vmbr1
iface vmbr1 inet static
        address 192.168.100.1/24
        bridge-ports none
        bridge-stp off
        bridge-fd 0
        post-up echo 1 > /proc/sys/net/ipv4/ip_forward
        post-up iptables -t nat -A POSTROUTING -s 192.168.100.0/24 -o vmbr0 -j MASQUERADE
        post-up /root/dnat.sh
        post-down iptables -t nat -D POSTROUTING -s 192.168.100.0/24 -o vmbr0 -j MASQUERADE

iface vmbr1 inet6 static
        address fde8:b429:841e:b651::1/64
        post-up echo 1 > /proc/sys/net/ipv6/conf/all/forwarding
        post-up ip6tables -t nat -A POSTROUTING -s fde8:b429:841e:b651::1/64 -o vmbr0 -j MASQUERADE
        post-down ip6tables -t nat -D POSTROUTING -s fde8:b429:841e:b651::1/64 -o vmbr0 -j MASQUERADE
```