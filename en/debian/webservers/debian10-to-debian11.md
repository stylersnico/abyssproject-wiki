---
title: Upgrade from Debian 10 to Debian 11
description: Upgrade from Buster to Bullseye
published: true
date: 2021-08-23T15:54:02.654Z
tags: debian
editor: markdown
dateCreated: 2021-08-12T17:37:50.044Z
---

# Before starting

The goal here is to upgrade an existing Debian 10 installation to Debian 11


# Updating actual system

First, update your system : 

```bash
apt update && apt dist-upgrade -y
```


# Sources update

Launch the following command to edit your existing sources and move to Debian 11's repository:

```bash
sed -i 's/buster/bullseye/g' /etc/apt/sources.list
```


# Upgrade to Debian 11

Finally, launch the following command to upgrade to Debian 11:

```bash
apt update && apt full-upgrade -y
reboot
```

## Error with Debian Security

> Error get on a CX11 server with the Debian 10 image at Hetzner
{.is-info}


If you get the following error:

```bash
E: The repository 'http://security.debian.org bullseye/updates Release' does not have a Release file.
N: Updating from such a repository can't be done securely, and is therefore disabled by default.
```

Replace this: 
```bash
deb http://security.debian.org/ bullseye/updates main contrib non-free
# deb-src http://security.debian.org/ bullseye/updates main contrib non-free
```

With this: 

```bash
deb http://security.debian.org/debian-security bullseye-security/updates main contrib non-free
# deb-src http://security.debian.org/debian-security bullseye-security/updates main contrib non-free
```