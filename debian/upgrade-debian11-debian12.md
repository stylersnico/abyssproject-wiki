---
title: Upgrade de Debian 11 vers Debian 12
description: Mise à jour de Debian Bullseye vers Debian Bookworm
published: true
date: 2023-07-10T12:55:16.195Z
tags: debian, debian 11, debian 12
editor: markdown
dateCreated: 2023-07-10T12:55:16.195Z
---

# Introduction
Le but de cet article est de migrer une installation de Debian 11 à Debian 12.

# Mise à jour du serveur

 
Commencez par mettre à jour votre serveur :

```bash
apt update 
apt full-upgrade
```

# Modifications des listes 

Mettez à jour vos listes d'installation avec les commandes suivantes :

```bash
sed -i 's/bullseye/bookworm/g' /etc/apt/sources.list
sed -i 's/non-free/non-free non-free-firmware/g' /etc/apt/sources.list #En cas d'utilisation des non-free
```

Le fichier complet ressemble à cela sans les non free :
```bash
deb http://deb.debian.org/debian/ bookworm main
deb-src http://deb.debian.org/debian/ bookworm main

deb http://security.debian.org/debian-security bookworm-security/updates main
deb-src http://security.debian.org/debian-security bookworm-security/updates main

# bookworm-updates, previously known as 'volatile'
deb http://deb.debian.org/debian/ bookworm-updates main
deb-src http://deb.debian.org/debian/ bookworm-updates main
```

# Upgrade du serveur

Lancez maintenant l'upgrade du serveur : 
```bash
apt update
apt full-upgrade
```

À la fin, vous devrez redémarrer :
```bash
reboot
```
