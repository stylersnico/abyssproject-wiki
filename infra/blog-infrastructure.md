---
title: Mon infrastructure personnelle
description: L'infrastructure chez moi et en dehors de chez moi
published: false
date: 2021-08-25T13:56:23.711Z
tags: 
editor: markdown
dateCreated: 2021-08-24T16:01:35.860Z
---

# Introduction
Le but de cette page est de vous présenter mon infrastructure et que vous voyez ce qui se cache derrière mes sites web.


# L'infrastructure du blog

## Serveur

Le serveur utilisé est un VPS CX11 de l'hébergeur allemand Hetzner sous Debian 11.
Le serveur est situé dans leur datacenter de Nuremberg.

Je dispose également d'une StorageBox pour les sauvegardes du blog.

## Stack technique

- Serveur Web / Reverse Proxy : NGINX (https://github.com/stylersnico/nginx-openssl-chacha-naxsi)
- Serveur SQL : MariaDB 10.5
- Wordpress : CMS mon blog : https://www.abyssproject.net/
- Grav : CMS pour mon site principal : https://www.nicolas-simond.ch/
- Wiki.JS : CMS pour le wiki (celui sur lequel vous êtes :))
- Commento : Système de commentaire pour le wiki
- Docker : Pour l'hébergement de Wiki.JS, Commento et leurs bases PostgreSQL respectives.
- Acme.SH : Pour les certificats SSL : https://wiki.abyssproject.net/fr/debian/webservers/acme_dot_sh-nginx
- Restic : Pour les sauvegardes


# L'infrastructre hors blog

## Serveurs

Je dipose d'un serveur de stockage chez OneProvider à Paris pour externaliser une copie de ma StorageBox Hetzner.
Je dispose d'un serveur chez Time4Vps pour un serveur OpenVPN.


# L'infrastucture @ home

## Serveur de virtualisation et stack technique

Le serveur est un Supermicro X10SDV-4C-TLN4F : https://www.supermicro.com/en/products/motherboard/X10SDV-4C-TLN4F.
Le stockage est le suivant : 
- 1x Samsung 870 evo NVME pour l'OS
- 2x Samsung 850 evo pour les datas
- 1x WD Red 6to pour les films et les gros fichiers

### Stack technique

- Virtualisation : Proxmox
- Données : Raid 1 avec XFS
- Machines virtuelles : QEMU avec drivers VirtIO
- Extinction automatique via Nuts en cas de coupure de courant

### Machines virtuelles

- Adguard : Hébergement de la solution ADguard de blocage des pubs et malwares via DNS.
- Ansible : Serveur Ansible de contrôle de l'infrastructure
- Bitwarden : Hébergement d'un serveur Bitwarden local
- LibreNMS : Monitoring de l'infrastructure
- Media : Serveur PLEX
- OPNSense : Serveur OPNSense virtualisé pour le firewalling
- Proton-backup : Serveur de backup de mes emails prontonmail
- Reverse : Reverse proxy nginx pour accès à mes services
- ZoneMinder : Machine pour la surveillance vidéo



## Serveur de backup et stack technique

Le serveur est un HP Proliant Microserver GEN8.

Le stockage est le suivant : 
- 1x Samsung 850 evo pour l'OS
- 4x WD Red 4to

### Stack technique

- Virtualisation : Hyper-V 2019
- Données : Raid 5 HP pour les données
- Veeam Backup & Replication pour export et chiffrement des sauvegardes PBS sur un volume Veracrypt externe  en rotation.

### Machines virtuelles

- PBS : Proxmox Backup Server virtualisé avec 3To d'espace disque pour la sauvegarde des machines virtuelles.