---
title: My personal infrastructure
description: My servers at home and on the internet
published: true
date: 2023-12-07T08:11:21.061Z
tags: selfhosting
editor: markdown
dateCreated: 2021-08-25T14:14:46.868Z
---

# Introduction
The goal of this page is to show you all the servers that I use so you can know what is behind my websites.


# Blog infrastructure

## Server

The server is a Kimsufi KS-LE-2 at French hoster OVH.

Server runs Proxmox and all data are inside Debian 11 virtual machines.

The backup are done on a VPS STOR-2 at Pulseheberg who is running Proxmox Backup Server and replicate backups to a Hetzner storage box.


## Technical stack

- Web server / Reverse Proxy: NGINX (https://github.com/stylersnico/nginx-secure-config/)
- SQL server: MariaDB 10.5
- Wordpress: CMS for my blog: https://www.abyssproject.net/
- Grav: CMS for my website: https://www.nicolas-simond.ch/
- Wiki.JS: CMS for the wiki (You are here :))
- Commento: Comment system for the wiki
- Docker: For hosting Wiki.JS, Commento and their PostgreSQL databases
- Acme.SH: For the SSL certificates: https://wiki.abyssproject.net/en/debian/webservers/acme_dot_sh-nginx
- Restic: For the backups
- Crowdsec : For security and log analysis
- UFW : Firewall


# @Home infrastructure

## Virtualisation server and technical stack
My main server is a supermicro X10SDV-4C-TLN4F: https://www.supermicro.com/en/products/motherboard/X10SDV-4C-TLN4F.
The storage is the following: 
- 1x Samsung 870 evo NVME 250Go for the OS
- 2x Samsung 850 evo 500Go for the datas
- 2x Seagate Barracuda 2.5" 5To for the movies and the big files

### Technical stack

- Virtualisation : Proxmox
- Data : Raid 1 with XFS
- Virtual machines : QEMU with VirtIO drivers
- Automatic shutdown with Nuts in case of power outage

### Virtual machines

- Adguard : Ad guard solution for blocking AD's and malware at DNS level
- Ansible : Ansible master to manage the infrastructure
- Bitwarden : Local Bitwarden hosting
- LibreNMS : Monitoring
- Media : PLEX server
- OPNSense : Virtualized OPNSense for firewalling
- Proton-backup : Automated backup of my prontonmail emails
- Reverse : NGINX Reverse proxy to access my services
- ZoneMinder : Videosurveillance system



## Backup server and technical stack

This server is an HP Proliant Microserver GEN8.

The storage is the following: 
- 1x Samsung 850 evo for the OS
- 4x Seagate Archive 8To

### Technical stack

- Proxmox Backup Server
- Data : Raid 10 ZFS for the data


## Internet connection

By internet connection is provided by Starlink.
The configuration is explained here: 
- https://wiki.abyssproject.net/en/starlink/connecting-starlink-opnsense
- https://wiki.abyssproject.net/en/starlink/port-opening-behind-starlink-purevpn