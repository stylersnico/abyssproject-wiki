---
title: Mon infrastructure personnelle
description: L'infrastructure chez moi et en dehors de chez moi
published: true
date: 2024-11-02T16:22:34.877Z
tags: selfhosting
editor: markdown
dateCreated: 2021-08-24T16:01:35.860Z
---

# Introduction
Le but de cette page est de vous présenter mon infrastructure et que vous voyez ce qui se cache derrière mes sites web.


# L'infrastructure

## Serveur

Le serveur est une machine virtuelle Debian 12 sur mon hyperviseur local.

## Stack technique

- Serveur Web / Reverse Proxy : NGINX (https://github.com/stylersnico/nginx-secure-config/)
- Serveur SQL : MariaDB
- Wordpress : CMS mon blog : https://www.abyssproject.net/
- Mon site principal en HTML Statique : https://www.nicolas-simond.ch/
- Wiki.JS : CMS pour le wiki (celui sur lequel vous êtes :))
- Docker : Pour l'hébergement de Wiki.JS et sa base PostgresSQL
- Crowdsec : Pour la sécurité et l'analyse des logs
- Cloudflared pour le lien avec Cloudflare et l'accès aux sites


# L'infrastucture @ home

## Serveur de virtualisation et stack technique

Le serveur est un Supermicro X10SDV-4C-TLN4F : https://www.supermicro.com/en/products/motherboard/X10SDV-4C-TLN4F.
Le stockage est le suivant : 
- 2x Firecuda 530 Nvme 1To pour l'OS et les machines virtuelles
- 2x Samsung 850 evo 500Go pour les machines de test
- 2x Seagate Barracuda 2.5" 5To pour les films et les gros fichiers

### Stack technique

- Virtualisation : Proxmox
- Données : Raid 1 avec BTRFS
- Machines virtuelles : QEMU avec drivers VirtIO
- Extinction automatique via Nuts en cas de coupure de courant

### Machines virtuelles

- Adguard : Hébergement de la solution ADguard de blocage des pubs et malwares via DNS.
- Ansible : Serveur Ansible de contrôle de l'infrastructure
- Bitwarden : Hébergement d'un serveur Bitwarden local
- LibreNMS : Monitoring de l'infrastructure
- Media : Serveur Jellyfin
- Proton-backup : Serveur de backup de mes emails prontonmail
- VPN : Mon VPN distant
- Webhost : Le serveur web

Avec ceci, j'ai un mini-routeur en Intel N100 et 4*2,5gbps pour l'OPNSense.

## Serveur de backup et stack technique

Le serveur est un HP Proliant Microserver GEN8.

Le stockage est le suivant : 
- 1x Samsung 850 evo pour l'OS
- 4x Seagate Archive 8To

### Stack technique

- Windows Server 2022 Datacenter et Veeam 
- Données : Raid 10 hardware


## Arrivée internet

Mon arrivée internet est fournie par Starlink.