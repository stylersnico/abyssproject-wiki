---
title: Upgrade Debian 12 to Debian 13
description: Upgrading Debian Bookworm to Debian Trixie
published: true
date: 2025-12-04T07:59:41.319Z
tags: debian
editor: markdown
dateCreated: 2025-12-04T07:59:41.319Z
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
sed -i 's/bookworm/trixie/g' /etc/apt/sources.list
```

The list looks like this without non-free:
```bash
deb http://deb.debian.org/debian/ trixie main
deb-src http://deb.debian.org/debian/ trixie main

deb http://security.debian.org/debian-security trixie-security/updates main
deb-src http://security.debian.org/debian-security trixie-security/updates main

# trixie-updates, previously known as 'volatile'
deb http://deb.debian.org/debian/ trixie-updates main
deb-src http://deb.debian.org/debian/ trixie-updates main
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
