---
title: Utiliser Proxmox avec une adresse ip publique
description: Utilisation de Proxmox chez Kimsufi, Hetzner, avec ouverture des ports pour les VMs et IPv6
published: false
date: 2023-02-20T14:28:23.445Z
tags: debian, hetzner, proxmox, kimsufi
editor: markdown
dateCreated: 2023-02-20T13:29:53.546Z
---

# Introduction

Le but de cet article est de mettre en place un Proxmox chez un hébergeur ne proposant qu'une unique IP (v4 et/ou v6) disponible sur un serveur dédié.
Le but sera que le machines virtuelles disposent d'internet et qu'il soit possible de redirigier des ports en local vers des services derrières les machines virtuelles.



# Pré-requis

 
Votre serveur doit déjà être installer avec la dernière version de Proxmox et l'adresse IP doit être configurée statiquement comme ceci par exemple : 

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

La communication se fera de cette façon : **VMBR0 <-> VMBR1 <-> Machines virtuelles**.

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

Le proxmox va évidemment gérer une partie NAT.


# Configuration de UFW

Pour la protection du Proxmox, je vais installer UFW afin de protéger un minimum l'hyperviseur (ce n'est pas obligatoire mais, faites-le).

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

Désactivez UFW : 
```bash
ufw disable
```

Ensuite, désactivez le pare-feu :
```bash
ufw disable
```

Autorisez toutes les connexions sortantes et refusez toutes les connexions entrantes :
```bash
ufw default deny incoming
ufw default allow outgoing
```

Si vous disposez d'une adresse IP Fixe, vous pouvez autoriser toutes les connexion depuis celle-ci, par exemple :
```bash
ufw allow from 45.23.28.24
```

Si vous n'avez pas encore d'ip fixe, autorisez juste le port 22 et le port de proxmox : 
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

Ouvrez l'interface web et de proxmox et créez votre nouvelle interface depuis Proxmox (**System** -> **Network** -> **Create** -> **Linux Bridge**) :

![proxmox-with-one-public-ip-01.png](/proxmox/proxmox-with-one-public-ip/proxmox-with-one-public-ip-01.png)

Remplissez l'interface comme ceci avec les réseaux privés que nous avons vu avant : 

![proxmox-with-one-public-ip-02.png](/proxmox/proxmox-with-one-public-ip/proxmox-with-one-public-ip-02.png)

> Vous remarquerez que je ne mets pas de **Bridge Port**.
> Si vous mettez une machine virtuelle ou un switch virtuel directement sur votre **VMBR0** sans avoir les adresses IP additionelles il est possible que votre port réseau soit automatiquement coupé au Datacenter.
{.is-danger}


# Configuration du routage

Nous allons configurer le **MASQUERADE** sous Linux.
Pour faire très simple, le **MASQUERADE** est un **NAT de type 1-to-many**.

Derrière cette explication sauvage se cache en fait le type de NAT commun derrière votre box ou n'importe quelle pare-feu.
Tous les PC du réseau internet peuvent sortir sur internet avec une seule adresse publique.

Ouvrez votre fichier d'interface :
```bash
nano /etc/network/interfaces
```

Ajoutez les lignes suivantes sur la carte VMBR1 en IPv4 :
```bash
post-up echo 1 > /proc/sys/net/ipv4/ip_forward
post-up iptables -t nat -A POSTROUTING -s 192.168.100.0/24 -o vmbr0 -j MASQUERADE
post-down iptables -t nat -D POSTROUTING -s 192.168.100.0/24 -o vmbr0 -j MASQUERADE
```

Ajoutez les lignes suivantes sur la carte VMBR1 en IPv6 : 
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

Dès cet instant, **redémarrez** vos machines virtuelles pourront avoir internet si vous mettez des adresses IP fixes sur leurs cartes réseaux.


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

# Ouverture des ports sur les machines virtuelles

Ici on va créer un script pour ouvrir les ports selon le modèle de NAT 1-to-1 (ouverture de ports classiques).

Dans notre exemple le port 80 en IPv4 et IPv6 


Créez un script pour rentrer vos configurations :
```bash
nano /root/dnat.sh
```

Mettez 
```bash
iptables -t nat -A PREROUTING -i vmbr0 -p tcp --dport 80 -j DNAT --to-destination 192.168.100.104:80
ip6tables -t nat -A PREROUTING -i vmbr0 -p tcp -m tcp --dport 80 -j DNAT --to-destination [fde8:b429:841e:b651::104]:80
```