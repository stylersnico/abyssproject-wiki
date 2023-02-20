---
title: Utiliser Proxmox avec une adresse ip publique
description: Utilisation de Proxmox chez Kimsufi, Hetzner, avec ouverture des ports pour les VMs et IPv6
published: false
date: 2023-02-20T13:29:53.546Z
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


# Configuration du Proxmox


