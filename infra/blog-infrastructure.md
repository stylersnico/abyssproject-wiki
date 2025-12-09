---
title: Mon infrastructure personnelle
description: L'infrastructure chez moi et en dehors de chez moi
published: true
date: 2025-12-09T08:10:59.833Z
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

- Serveur Web : NGINX (https://github.com/stylersnico/nginx-secure-config/)
- Serveur SQL : MariaDB
- Wordpress : CMS mon blog : https://www.abyssproject.net/
- Mon site principal en HTML Statique : https://www.nicolas-simond.ch/
- Wiki.JS : CMS pour le wiki (celui sur lequel vous êtes :))
- Docker : Pour l'hébergement de Wiki.JS et sa base PostgresSQL
- Wazuh : Pour la sécurité et l'analyse des logs
- Cloudflared : Pour le lien avec Cloudflare et l'accès aux sites


# L'infrastucture @ home

## Serveur de virtualisation et stack technique

Le serveur est un Supermicro X10SDV-4C-TLN4F : https://www.supermicro.com/en/products/motherboard/X10SDV-4C-TLN4F.
Le stockage est le suivant : 
- 2x Firecuda 530 Nvme 1To pour l'OS et les machines virtuelles
- 2x Seagate Exos X16 14To pour les films et les gros fichiers

### Stack technique

- Virtualisation : Windows Server 2022 Datacenter et Hyper-V
- Données : Raid 1
- Extinction automatique via Powerchute en cas de coupure de courant

### Machines virtuelles

Sauf indication contraire, tout est sous Debian 12.

- Ansible : Serveur Ansible de contrôle de l'infrastructure
- LibreNMS : Monitoring de l'infrastructure
- HAOS : Serveur Home Assistant
- Paperless : GED paperless-ngx
- Media : Serveur Jellyfin
- Passbolt : Gestionnaire de mots de passes
- Proton-backup : Serveur de backup de mes emails prontonmail
- Wazuh : SIEM / XDR
- Webhost : Le serveur web

Avec ceci, j'ai un mini-routeur en Intel N100 et 4*2,5gbps pour l'OPNSense.


## Serveur de backup et stack technique

Le serveur est un HP Proliant Microserver GEN8.

Le stockage est le suivant : 
- 2x WD Red SA500 500Go pour l'OS
- 2x Seagate Exos X16 16To

### Stack technique

- Windows Server 2022 Datacenter et Veeam 
- Données : Raid 1


## Arrivée internet

Mon arrivée internet est fournie par Orange.
La ligne de secours est assurée par Starlink.