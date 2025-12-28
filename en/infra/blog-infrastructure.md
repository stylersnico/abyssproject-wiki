---
title: My personal infrastructure
description: My servers at home and on the internet
published: true
date: 2025-12-28T09:18:30.861Z
tags: selfhosting
editor: markdown
dateCreated: 2021-08-25T14:14:46.868Z
---

# Introduction
The goal of this page is to show you all the servers that I use so you can know what is behind my websites.


# Blog infrastructure

## Server

Server runs inside a Debian 13 virtual machine.


## Technical stack

- Web server / Reverse Proxy: NGINX (https://github.com/stylersnico/nginx-secure-config/)
- SQL server: MariaDB
- Wordpress: CMS for my blog: https://www.abyssproject.net/
- My website in pure HTML: https://www.nicolas-simond.ch/
- Wiki.JS: CMS for the wiki (You are here :))
- Docker: For hosting Wiki.JS and psql database
- Wazuh: For security and log analysis
- Cloudflared: For remote tunnel and access with cloudflare


# @Home infrastructure

## Virtualisation server and technical stack
My main server is a supermicro X10SDV-4C-TLN4F: https://www.supermicro.com/en/products/motherboard/X10SDV-4C-TLN4F.
The storage is the following: 
- 2x Firecuda 530 1To for the OS and VMs (with the new native Windows NVME driver)
- 2x Seagate Exos X16 14To for the movies and the big files

### Technical stack

- Virtualisation : Windows Server 2025 Datacenter and Hyper-V
- DATA : Raid 1
- Powerchute shutdown in case of power outage

### Virtual machines
Everything is under Debian 13.

- Ansible : Infrastructure management
- LibreNMS : Monitoring
- HAOS : Home Assistant server
- Paperless : paperless-ngx edm software
- Media : Jellyfin (Debian 12)
- Passbolt : Password manager
- Proton-backup : Protonmail backup server
- Wazuh : SIEM / XDR (Ubuntu 24.04)
- Webhost : Web server

I also have a small router with Intel N100 and 4*2,5Gbps for OPNSense.

## Backup server and technical stack

This server is an HP Proliant Microserver GEN8.

The storage is the following: 
- 2x WD Red SA500 500Go
- 2x Seagate Exos X16 16To

### Technical stack

- Windows Server 2025 Datacenter with Veeam
- Data : Hardware Raid 1 for the data


## Internet connection

My internet connection is provided by Orange.
Backup connection is provided by Starlink.