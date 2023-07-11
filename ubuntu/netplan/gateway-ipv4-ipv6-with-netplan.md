---
title: Gateway IPv4 + IPv6 avec Netplan
description: Comment avoir la passerelle par défaut en IPv4 + IPv6 avec Netplan
published: true
date: 2023-07-11T14:17:04.058Z
tags: netplan, ubuntu, ipv6, ipv4
editor: markdown
dateCreated: 2023-07-11T14:17:04.058Z
---

# Introduction

Le but de cet article est d'avoir une passerelle par défaut IPv4 et une passerelle par défaut IPv6 simultanément via la configuration d'interface de Netplan.


# Configuration des passerelles

Ouvrez votre fichier de configuration des interfaces, chez moi par exemple : 
```bash
nano /etc/netplan/00-installer-config.yaml
```
 
Ajoutez les routes de la façon suivante : 
```bash
      routes:
      - to: default
        via: 192.168.100.1
      - to: "::/0"
        via: fde8:b429:841e:b651::1
```

Voici mon fichier complet par exemple : 

```bash
network:
  ethernets:
    ens18:
      addresses:
      - 192.168.100.100/24
      - fde8:b429:841e:b651::100/64
      nameservers:
        addresses:
        - 1.1.1.1
        - 8.8.8.8
        - 2606:4700:4700::1111
        - 2606:4700:4700::1001
        search: []
      routes:
      - to: default
        via: 192.168.100.1
      - to: "::/0"
        via: fde8:b429:841e:b651::1
  version: 2
```

Ensuite, testez votre configuration avec la commande suivante : 

```bash
netplan try -timeout 30
```

Si la configuration est bonne, appliquez là ou redémarrez le serveur : 

```bash
netplan apply
```