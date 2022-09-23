---
title: Ouverture de ports sur une connexion Starlink
description: Ouverture de ports sur une connexion Starlink via PureVPN
published: false
date: 2022-09-23T12:34:44.967Z
tags: starlink, purevpn
editor: markdown
dateCreated: 2022-09-23T12:34:44.967Z
---

# Introduction
Le but de cette procédure est de pouvoir ouvrir des ports en entrée sur une connexion Starlink via un prestataire VPN externe comme PureVPN dans cet exemple.

> Dans cet exemple, PureVPN sera utilisé, il n'y aucune affiliation avec eux.
> Vous devez avoir l'abonnement avec l'add-on **Port Forwarding** : https://www.purevpn.com/port-forwarding
{.is-info}

Vous devez également avoir un firewall comme OPNSense sur votre connexion Starlink qui servira déjà de firewall évidemment et ensuite permettra de faire du nat uniquement sur les ports souhaités.

Exemple de configuration d'un firewall OPNSense avec Starlink : https://wiki.abyssproject.net/fr/starlink/connecting-starlink-opnsense


# Configuration de PureVPN

Ouvrez votre souscription chez PureVPN et configurez le port Forwarding : https://my.purevpn.com/v2/dashboard/subscriptions

![port-opening-behind-starlink-purevpn-01.png](/starlink/nat-behind-starlink/port-opening-behind-starlink-purevpn-01.png)


Activez-tous les ports, ceci est uniquement sécurisé si vous utilisez un firewall comme OPNSense en tant que client, sinon ne faites pas cela.

![port-opening-behind-starlink-purevpn-02.png](/starlink/nat-behind-starlink/port-opening-behind-starlink-purevpn-02.png)


# Configuration du firewall OPNSense

## Configuration du client OpenVPN

Téléchargez les derniers fichiers de configuration depuis ici (**Others routers**) : https://support.purevpn.com/openvpn-files.

Allez dans **System** -> **Trust** -> **Authorities** et cliquez sur le **+** :

![port-opening-behind-starlink-purevpn-03.png](/starlink/nat-behind-starlink/port-opening-behind-starlink-purevpn-03.png)

Importez le certificat d'autorité présent dans l'archive (le contenu du fichier **ca2.crt**) :

![port-opening-behind-starlink-purevpn-04.png](/starlink/nat-behind-starlink/port-opening-behind-starlink-purevpn-04.png)

Maintenant, allez dans **VPN** -> **OpenVPN** -> **Clients** et cliquez sur le **+** :
![port-opening-behind-starlink-purevpn-05.png](/starlink/nat-behind-starlink/port-opening-behind-starlink-purevpn-05.png)


> Votre nom d'utilisateur et votre mot de passe PureVPN sont disponible en bas de cette page : https://my.purevpn.com/v2/dashboard/subscriptions
{.is-info}


Vous pourez également trouver les informations du serveur dans le fichier de configuration OpenVPN que vous souhaitez utiliser, par exemple dans **fr2-ovpn-udp.ovpn** :

![port-opening-behind-starlink-purevpn-07.png](/starlink/nat-behind-starlink/port-opening-behind-starlink-purevpn-07.png)

Remplissez maintenant toutes les informations importantes comme ceci, vous aurez besoin du contenu du fichier **wdc.key** :

![port-opening-behind-starlink-purevpn-06.png](/starlink/nat-behind-starlink/port-opening-behind-starlink-purevpn-06.png)

Allez dans **VPN** -> **OpenVPN** -> **Connection Status** et vérifiez que la connexion est OK :

![port-opening-behind-starlink-purevpn-08.png](/starlink/nat-behind-starlink/port-opening-behind-starlink-purevpn-08.png)


## Configuration de l'interface virtuelle OpenVPN :

Allez dans **Intefaces** -> **Assignments** et asignez l'interface client OpenVPN sur OPT1 (ou la première interface disponible) :
![port-opening-behind-starlink-purevpn-09.png](/starlink/nat-behind-starlink/port-opening-behind-starlink-purevpn-09.png)


Configurez uniquement la description et activez l'interface : 
![port-opening-behind-starlink-purevpn-10.png](/starlink/nat-behind-starlink/port-opening-behind-starlink-purevpn-10.png)


## Configuration du routage

## Sources
- M4DM4NZ : https://forum.opnsense.org/index.php?topic=4979.0