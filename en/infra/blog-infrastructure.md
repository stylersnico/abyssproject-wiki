---
title: My personal infrastructure
description: My servers at home and on the internet
published: true
date: 2021-08-31T09:23:12.282Z
tags: selfhosting
editor: markdown
dateCreated: 2021-08-25T14:14:46.868Z
---

# Introduction
The goal of this page is to show you all the servers that I use so you can know what is behind my websites.


# Blog infrastructure

## Server

The server used is an Hetzner's CX11 installed with Debian 11.
The server is located in their datacenter in Nuremberg.

I also have a StorageBox for the blog.

## Technical stack

- Web server / Reverse Proxy: NGINX (https://github.com/stylersnico/nginx-openssl-chacha-naxsi)
- SQL server: MariaDB 10.5
- Wordpress: CMS for my blog: https://www.abyssproject.net/
- Grav: CMS for my website: https://www.nicolas-simond.ch/
- Wiki.JS: CMS for the wiki (You are here :))
- Commento: Comment system for the wiki
- Docker: For hosting Wiki.JS, Commento and their PostgreSQL databases
- Acme.SH: For the SSL certificates: https://wiki.abyssproject.net/en/debian/webservers/acme_dot_sh-nginx
- Restic: For the backups


# Non-blog infrastructre

## Servers

I have one storage server with OneProvider in Paris to externalize a copy of all my storagebox content at Hetzner.
I also have a server with Time4VPS to use as a VPN.

# @Home infrastructure

## Virtualisation server and technical stack
My main server is a supermicro X10SDV-4C-TLN4F: https://www.supermicro.com/en/products/motherboard/X10SDV-4C-TLN4F.
The storage is the following: 
- 1x Samsung 870 evo NVME for the OS
- 2x Samsung 850 evo for the datas
- 1x WD Red 6to for the movies and the big files

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
- 4x WD Red 4to

### Technical stack

- Virtualisation : Hyper-V 2019
- Data : Raid 5 HP for the datas
- Veeam Backup & Replication for the export and the encryption of all the data backed up by PBS on an external veracrypt rotated repository.

### Virtual Machines

- PBS : Virtualized Proxmox Backup Server with 3Tb of disk storage for the backup of the virtual machines.