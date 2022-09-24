---
title: Connexion de Starlink à OPNSense
description: Connexion de Starlink à OPNSense en mode bridge
published: true
date: 2022-09-24T13:57:07.888Z
tags: opnsense, starlink, bridge
editor: markdown
dateCreated: 2022-09-23T11:34:44.005Z
---

# Introduction
Le but de cette procédure est de connecter le routeur Starlink en mode bridge à votre pare-feu OPNsense.

> Vous aurez besoin de l'adaptateur RJ45 Starlink si vous ne l'avez pas déjà afin de mettre votre routeur en mode Bridge : https://shop.starlink.com/products/fr-consumer-ethernet-adapter-gen2
{.is-warning}


![starlink-ethernet.jpg](/starlink/starlink-ethernet.jpg)

# Configuration du mode Bridge sur le routeur Starlink

Connectez-vous au réseau sans fil par défaut et ouvrez l'application Starlink depuis votre téléphone.
Maintenant, allez dans **Settings** et activez le paramètre **Bypass Starlink WiFi router** :

![connecting-starlink-opnsense-01.png](/starlink/connecting-starlink-opnsense-01.png)

Attendez 5 minutes que le routeur redémarre et connectez le câble ethernet à l'interface **WAN** de votre firewall OPNSense.


# Configuration de OPNSense

Allez dans **Interfaces** -> **Wan** et configurez l'interface comme ceci via **DHCPv4**.
> Décochez impérativement **Block private networks** sinon vous n'obtiendrez aucune adresse vu que starlink vous fournira une adresse IP sur un range pour du Carrier Grade Nat (100.64/10).
{.is-warning}


![connecting-starlink-opnsense-02.png](/starlink/connecting-starlink-opnsense-02.png)

Sauvegardez les paramètres de l'interface.


Maintenant, allez dans **System** -> **Gateways** -> **Single** et modifiez la passerelle par défaut sur l'interface **WAN** :

![connecting-starlink-opnsense-03.png](/starlink/connecting-starlink-opnsense-03.png)

Mettez l'adresse IP d'un DNS publique, par exemple, en monitoring plutôt que la passerelle fournie par Starlink afin d'avoir un monitoring cohérent (la passerelle par défaut sera toujours UP comme elle est en local logiquement) :

![connecting-starlink-opnsense-04.png](/starlink/connecting-starlink-opnsense-04.png)


Vous pourrez enfin profiter de votre accès internet :

![connecting-starlink-opnsense-05.png](/starlink/connecting-starlink-opnsense-05.png)