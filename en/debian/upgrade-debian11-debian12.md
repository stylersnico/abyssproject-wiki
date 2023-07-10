---
title: Upgrade Debian 11 to Debian 12
description: Upgrading Debian Bullseye to Debian Bookworm
published: true
date: 2023-07-10T12:59:52.274Z
tags: debian, debian 11, debian 12
editor: markdown
dateCreated: 2023-07-10T12:59:52.274Z
---

# Introduction
The goal of this guide is to migrate Debian 11 to Debian 12.

# Updating the server
 
First, update your server:

```bash
apt update 
apt full-upgrade
```

# Editing sources.list

Update your source file with the following command:

```bash
sed -i 's/bullseye/bookworm/g' /etc/apt/sources.list
sed -i 's/non-free/non-free non-free-firmware/g' /etc/apt/sources.list #En cas d'utilisation des non-free
```

The list looks like this without non-free:
```bash
deb http://deb.debian.org/debian/ bookworm main
deb-src http://deb.debian.org/debian/ bookworm main

deb http://security.debian.org/debian-security bookworm-security/updates main
deb-src http://security.debian.org/debian-security bookworm-security/updates main

# bookworm-updates, previously known as 'volatile'
deb http://deb.debian.org/debian/ bookworm-updates main
deb-src http://deb.debian.org/debian/ bookworm-updates main
```

# Upgrading the server

Now launch the full upgrade of the server: 
```bash
apt update
apt full-upgrade
```

At the end, reboot:
```bash
reboot
```
