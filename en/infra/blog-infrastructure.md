---
title: My personal infrastructure
description: My servers at home and on the internet
published: true
date: 2024-11-02T16:23:15.497Z
tags: selfhosting
editor: markdown
dateCreated: 2021-08-25T14:14:46.868Z
---

# Introduction
The goal of this page is to show you all the servers that I use so you can know what is behind my websites.


# Blog infrastructure

## Server

Server runs inside a Debian 12 virtual machine.


## Technical stack

- Web server / Reverse Proxy: NGINX (https://github.com/stylersnico/nginx-secure-config/)
- SQL server: MariaDB
- Wordpress: CMS for my blog: https://www.abyssproject.net/
- My website in pure HTML: https://www.nicolas-simond.ch/
- Wiki.JS: CMS for the wiki (You are here :))
- Commento: Comment system for the wiki
- Docker: For hosting Wiki.JS, Commento and their PostgreSQL databases
- Crowdsec: For security and log analysis
- Cloudflared: For remote tunnel and access with cloudflare


# @Home infrastructure

## Virtualisation server and technical stack
My main server is a supermicro X10SDV-4C-TLN4F: https://www.supermicro.com/en/products/motherboard/X10SDV-4C-TLN4F.
The storage is the following: 
- 2x Firecuda 530 1To for the OS and VMs
- 2x Samsung 850 evo 500Go for testing things
- 2x Seagate Barracuda 2.5" 5To for the movies and the big files

### Technical stack

- Virtualisation : Proxmox
- Data : Raid 1 with BTRFS
- Virtual machines : QEMU with VirtIO drivers
- Automatic shutdown with Nuts in case of power outage

### Virtual machines

- Adguard: Ad guard solution for blocking AD's and malware at DNS level
- Ansible: Ansible master to manage the infrastructure
- Bitwarden: Local Bitwarden hosting
- LibreNMS: Monitoring
- Media: Jellyfin server
- Proton-backup: Automated backup of my prontonmail emails
- Reverse: NGINX Reverse proxy to access my services
- Webhost: My webhost

I also have a small router with Intel N100 and 4*2,5Gbps for OPNSense.

## Backup server and technical stack

This server is an HP Proliant Microserver GEN8.

The storage is the following: 
- 1x Samsung 850 evo for the OS
- 4x Seagate Archive 8To

### Technical stack

- Windows Server 2022 Datacenter with Veeam
- Data : Hardware Raid 10 for the data


## Internet connection

By internet connection is provided by Starlink.