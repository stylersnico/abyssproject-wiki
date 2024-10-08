---
title: Utiliser Proxmox avec une adresse ip publique
description: Utilisation de Proxmox chez Kimsufi, Hetzner, avec ouverture des ports pour les VMs et IPv6
published: true
date: 2024-08-21T11:43:22.138Z
tags: debian, hetzner, proxmox, kimsufi
editor: markdown
dateCreated: 2023-02-20T13:29:53.546Z
---

# Introduction

Le but de cet article est de mettre en place un Proxmox chez un hébergeur ne proposant qu'une unique IP (v4 et/ou v6) disponible sur un serveur dédié.
Le but sera que les machines virtuelles disposent d'internet et qu'il soit possible de rediriger des ports en local vers des services derrière les machines virtuelles.

> Il est évident que le montage ci-dessous doit-être considéré comme un bricolage plus qu'une bonne pratique.
> Toutefois, c'est l'infrastructure qui héberge mon blog et mes services et pour un usage personnel cela fonctionne correctement.
{.is-warning}


# Pré-requis

 
Votre serveur doit déjà être installé avec la dernière version de Proxmox et l'adresse IP doit être configurée statiquement comme ceci par exemple : 

![proxmox-with-one-public-ip-00.png](/proxmox/proxmox-with-one-public-ip/proxmox-with-one-public-ip-00.png)


# Description du fonctionnement

Le réseau externe (VMBR0) dispose d'une adresse IPv4 et d'une adresse IPv6 disponible : 
- `Réseau IPv4 : 37.187.147.215/24`
- `Réseau IPv6 : 2001:41d0:a:53d7::1/128`

Avant tout, nous allons devoir faire un réseau interne.

Dans cet exemple, le réseau interne (VMBR1) disposera d'IPv4 et d'IPv6 : 
- `Réseau IPv4 : 192.168.100.1/24`
- `Réseau IPv6 : fde8:b429:841e:b651::1/64`

> Le range IPv6 est arbitraite et a été généré avec cet outil : https://simpledns.plus/private-ipv6.
Il est parfaitement valide pour l'usage qui va en être fait.
{.is-info}

La communication se fera ainsi : **VMBR0 <-> VMBR1 <-> Machines virtuelles**.

Les machines virtuelles auront des adresses dans le réseau interne (VMBR1), voici un exemple pour une machine virtuelle : 
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

Le Proxmox va évidemment gérer une partie NAT.


# Configuration de UFW

Pour la protection du Proxmox, on installe UFW afin de protéger un minimum l'hyperviseur (ce n'est pas obligatoire, mais faites-le).

Installez d'abord UFW : 
```bash
apt-get install ufw -y
```

Ouvrez ensuite le fichier de configuration par défaut : 
```bash
nano /etc/default/ufw
```

Configurez les lignes suivantes : 
```bash
IPV6=yes
DEFAULT_FORWARD_POLICY="ACCEPT"
```

Ouvrez ensuite le fichier de configuration suivant : 
```bash
nano /etc/ufw/sysctl.conf
```

Configurez les lignes suivantes : 
```bash
net/ipv4/ip_forward=1
net/ipv6/conf/default/forwarding=1
net/ipv6/conf/all/forwarding=1
```

Ensuite, désactivez le pare-feu :
```bash
ufw disable
```

Autorisez toutes les connexions sortantes et refusez toutes les connexions entrantes sauf sur le réseau interne :
```bash
ufw default deny incoming
ufw default allow outgoing
ufw allow in on vmbr1 to any
```

Si vous disposez d'une adresse IP Fixe, vous pouvez autoriser toutes les connexions depuis celle-ci, par exemple :
```bash
ufw allow from 45.23.28.24
```

Si vous n'avez pas encore d'IP fixe, autorisez juste le port 22 et le port de Proxmox : 
```bash
ufw allow 22
ufw allow 8006
```

Ensuite, activez le firewall : 
```bash
ufw enable
```

> Bonus, si vous souhaitez utiliser UFW avec une IP dynamique : https://www.abyssproject.net/2017/07/utiliser-ip-dynamique-nginx-ufw/
{.is-info}



# Configuration du réseau interne (VMBR1)

Ouvrez l'interface web et de Proxmox et créez votre nouvelle interface depuis Proxmox (**System** -> **Network** -> **Create** -> **Linux Bridge**) :

![proxmox-with-one-public-ip-01.png](/proxmox/proxmox-with-one-public-ip/proxmox-with-one-public-ip-01.png)

Remplissez l'interface comme ceci avec les réseaux privés que nous avons vus avant : 

![proxmox-with-one-public-ip-02.png](/proxmox/proxmox-with-one-public-ip/proxmox-with-one-public-ip-02.png)

> Vous remarquerez que je ne mets pas de **Bridge Port**.
> Si vous mettez une machine virtuelle ou un switch virtuel directement sur votre **VMBR0** sans avoir les adresses IP additionelles il est possible que votre port réseau soit automatiquement coupé au Datacenter.
{.is-danger}


# Configuration du routage

Nous allons configurer le **MASQUERADE** sous Linux.
Pour faire très simple, le **MASQUERADE** est un **NAT de type 1-to-many**.

Derrière cette explication sauvage se cache en fait le type de NAT commun derrière votre box ou n'importe quelle pare-feu.
Tous les PC du réseau interne peuvent sortir sur internet avec une seule adresse publique.

Ouvrez votre fichier d'interface :
```bash
nano /etc/network/interfaces
```

Ajoutez les lignes suivantes sur la carte **VMBR1** en **IPv4** :
```bash
post-up echo 1 > /proc/sys/net/ipv4/ip_forward
post-up iptables -t nat -A POSTROUTING -s 192.168.100.0/24 -o vmbr0 -j MASQUERADE
post-down iptables -t nat -D POSTROUTING -s 192.168.100.0/24 -o vmbr0 -j MASQUERADE
```

Ajoutez les lignes suivantes sur la carte **VMBR1** en **IPv6** : 
```bash
post-up echo 1 > /proc/sys/net/ipv6/conf/all/forwarding
post-up ip6tables -t nat -A POSTROUTING -s fde8:b429:841e:b651::1/64 -o vmbr0 -j MASQUERADE
post-down ip6tables -t nat -D POSTROUTING -s fde8:b429:841e:b651::1/64 -o vmbr0 -j MASQUERADE
```

De façon très concrète, votre fichier d'interface devrait ressembler à ceci : 

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

Dès cet instant, **redémarrez** votre Proxmox et vos machines virtuelles pourront avoir internet si vous mettez des adresses IP fixes sur leurs cartes réseaux.


# Configuration d'un DHCP pour les machines virtuelles

Pour simplifier l'installation de vos machines, vous pouvez faire un petit serveur DHCPv4 qui donnera un pool d'IP à vos machines virtuelles.

Installez DNSMasq : 

```bash
apt install dnsmasq -y
```

Ouvrez le fichier de configuration :
```bash
nano /etc/dnsmasq.conf
```

Configurez un range sur VMBR1 comme ceci :

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

Activez et lancez le service : 
```bash
systemctl enable dnsmasq && systemctl start dnsmasq
```

# Configuration d'une adresse IP fixe sur les machines virtuelles

Dans cet exemple, je prends le cas d'une machine virtuelle sous Debian 11 dont le réseau est sur le switch virtuel **VMBR1** :

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

# Ouverture des ports sur les machines virtuelles

Ici, on va créer un script pour ouvrir les ports voulus sur les VMs.

Dans notre exemple, le port 80 sera ouvert en IPv4 et IPv6 en redirigeant sur l'adresse IP **.104**.


Créez un script pour rentrer vos configurations :

```bash
nano /root/dnat.sh
```

Ouvrez, par exemple, le port 80 sur la machine virtuelle en .104 en IPv4 et en IPv6 : 

```bash
sleep 60
iptables -t nat -A PREROUTING -i vmbr0 -p tcp --dport 80 -j DNAT --to-destination 192.168.100.104:80
ip6tables -t nat -A PREROUTING -i vmbr0 -p tcp -m tcp --dport 80 -j DNAT --to-destination [fde8:b429:841e:b651::104]:80
```

> **iptables** gère les tables de NAT en **IPv4**.
> **ip6tables** gère les tables de NAT en **IPv6**.
{.is-info}


Attention, si vous avez configuré UFW, vous devrez aussi ouvrir le port : 

```bash
ufw allow 80
```

Ensuite, vous devez activer le script dans le fichier interface : 

```bash
nano /etc/network/interfaces
```

Ajoutez les lignes suivantes sur la carte VMBR1 en IPv4 :

```bash
post-up echo 1 > /proc/sys/net/ipv4/ip_forward
post-up iptables -t nat -A POSTROUTING -s 192.168.100.0/24 -o vmbr0 -j MASQUERADE
post-up /root/dnat.sh
post-down iptables -t nat -D POSTROUTING -s 192.168.100.0/24 -o vmbr0 -j MASQUERADE
```

Ainsi, le NAT sera appliqué à chaque redémarrage.

Voici l'exemple de fichier que j'utilise pour mes services : 

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

Mon fichier interface complet ressemble à ceci : 

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