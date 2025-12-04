---
title: Upgrade de Debian 12 vers Debian 13
description: Mise à jour de Debian Bookworm vers Debian Trixie
published: true
date: 2025-12-04T07:59:49.640Z
tags: debian, trixie
editor: markdown
dateCreated: 2025-12-04T07:58:18.146Z
---

# Introduction
Le but de cet article est de migrer une installation de Debian 12 à Debian 13.

# Mise à jour du serveur

 
Commencez par mettre à jour votre serveur :

```bash
apt update 
apt full-upgrade
```

# Modifications des listes 

Mettez à jour vos listes d'installation avec les commandes suivantes :

```bash
sed -i 's/bookworm/trixie/g' /etc/apt/sources.list
```

Le fichier complet ressemble à cela sans les non free :
```bash
deb http://deb.debian.org/debian/ trixie main
deb-src http://deb.debian.org/debian/ trixie main

deb http://security.debian.org/debian-security trixie-security/updates main
deb-src http://security.debian.org/debian-security trixie-security/updates main

# trixie-updates, previously known as 'volatile'
deb http://deb.debian.org/debian/ trixie-updates main
deb-src http://deb.debian.org/debian/ trixie-updates main
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
