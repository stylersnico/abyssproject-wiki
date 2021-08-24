---
title: Mise à jour de Debian 10 vers Debian 11
description: Upgrade de Buster à Bullseye
published: true
date: 2021-08-23T15:53:38.194Z
tags: 
editor: markdown
dateCreated: 2021-08-11T17:56:25.121Z
---

# Introduction

Le but de cet article est de mettre à jour un serveur Debian 10 existant vers Debian 11 


# Mise à jour du système existant

Dans un premier temps, mettez le système existant à jour : 

```bash
apt update && apt dist-upgrade -y
```


# Modification des sources

Lancez la commande suivante pour modifier les sources existantes et passer sur les repository de Debian 11 : 

```bash
sed -i 's/buster/bullseye/g' /etc/apt/sources.list
```


# Mise à jour vers Debian 11

Lancez les commandes suivantes pour passer sur Debian 11 : 

```bash
apt update && apt full-upgrade -y
reboot
```

## Erreur sur Debian Security

> Erreur rencontrée sur sur un CX11 avec l'image Debian 10 chez l'hébergeur Hetzner
{.is-info}


Si vous recevez l'erreur suivante :

```bash
E: The repository 'http://security.debian.org bullseye/updates Release' does not have a Release file.
N: Updating from such a repository can't be done securely, and is therefore disabled by default.
```

Remplacez ceci : 
```bash
deb http://security.debian.org/ bullseye/updates main contrib non-free
# deb-src http://security.debian.org/ bullseye/updates main contrib non-free
```

Par cela : 

```bash
deb http://security.debian.org/debian-security bullseye-security/updates main contrib non-free
# deb-src http://security.debian.org/debian-security bullseye-security/updates main contrib non-free
```