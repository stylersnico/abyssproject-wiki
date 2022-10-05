---
title: Ouverture de ports sur une connexion Starlink
description: Ouverture de ports sur une connexion Starlink via PureVPN
published: true
date: 2022-10-05T14:03:22.086Z
tags: opnsense, starlink, purevpn, nat
editor: markdown
dateCreated: 2022-09-23T12:34:44.967Z
---

> **Le service de PureVPN n'est pas bon**, je laisse ce guide en archive.
> Si vous voulez utiliser votre propre VPS pour faire un VPN, voici un guide : https://wiki.abyssproject.net/en/starlink/port-opening-behind-starlink-wireguard
{.is-danger}




# Introduction
Le but de cette procédure est de pouvoir ouvrir des ports en entrée sur une connexion Starlink via un prestataire VPN externe comme PureVPN dans cet exemple.

> Dans cet exemple, PureVPN sera utilisé, mais il n'y aucune affiliation avec eux.
> Vous devez avoir l'abonnement avec l'add-on **Port Forwarding** : https://www.purevpn.com/port-forwarding
{.is-info}

Vous devez également avoir un firewall comme OPNSense sur votre connexion Starlink qui servira déjà de firewall évidemment et ensuite permettra de faire du nat uniquement sur les ports souhaités.

Exemple de configuration d'un firewall OPNSense avec Starlink : https://wiki.abyssproject.net/fr/starlink/connecting-starlink-opnsense


# Configuration de PureVPN

Ouvrez votre souscription chez PureVPN et configurez le port Forwarding : https://my.purevpn.com/v2/dashboard/subscriptions

![port-opening-behind-starlink-purevpn-01.png](/starlink/nat-behind-starlink/port-opening-behind-starlink-purevpn-01.png)


Activez-tous les ports, ceci est uniquement sécurisé si vous utilisez un firewall comme OPNSense comme client, sinon ne faites pas cela.

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


Vous pourrez également trouver les informations du serveur dans le fichier de configuration OpenVPN que vous souhaitez utiliser, par exemple, dans **fr2-ovpn-udp.ovpn** :

![port-opening-behind-starlink-purevpn-07.png](/starlink/nat-behind-starlink/port-opening-behind-starlink-purevpn-07.png)

Remplissez maintenant toutes les informations importantes comme ceci, vous aurez besoin du contenu du fichier **wdc.key** dans le champ **TLS Shared Key** :

![port-opening-behind-starlink-purevpn-06.png](/starlink/nat-behind-starlink/port-opening-behind-starlink-purevpn-06.png)

Allez dans **VPN** -> **OpenVPN** -> **Connection Status** et vérifiez que la connexion est OK :

![port-opening-behind-starlink-purevpn-08.png](/starlink/nat-behind-starlink/port-opening-behind-starlink-purevpn-08.png)


## Configuration de l'interface virtuelle OpenVPN

Allez dans **Intefaces** -> **Assignments** et assignez l'interface client OpenVPN sur OPT1 (ou la première interface disponible) :
![port-opening-behind-starlink-purevpn-09.png](/starlink/nat-behind-starlink/port-opening-behind-starlink-purevpn-09.png)


Configurez uniquement la description et activez l'interface : 
![port-opening-behind-starlink-purevpn-10.png](/starlink/nat-behind-starlink/port-opening-behind-starlink-purevpn-10.png)


## Configuration du routage

Allez dans **Firewall** -> **Aliases** et créez un Alias qui regroupera les IP de vos machines qui doivent sortir sur Internet via le VPN nouvellement mis en place :

![port-opening-behind-starlink-purevpn-11.png](/starlink/nat-behind-starlink/port-opening-behind-starlink-purevpn-11.png)

Maintenant, allez dans **Firewall** -> **LAN** et créez les deux règles suivantes :

- La première règle permet à vos clients dans l'alias **PureVPNClients** de sortir et d'entrer via le tunnel VPN.
- La seconde règle laisse tout le reste de sortir via le **WAN** Starlink.

![port-opening-behind-starlink-purevpn-12.png](/starlink/nat-behind-starlink/port-opening-behind-starlink-purevpn-12.png)

## Configuration du NAT sortant

Allez dans **Firewall** -> **Outbound** et passez votre firewall en **Hybrid outbound NAT rule generation**.

Ensuite, créez les 3 règles suivantes (qui sont sur l'interface PureVPN) afin que le trafic sortant soit fonctionnel :

![port-opening-behind-starlink-purevpn-13.png](/starlink/nat-behind-starlink/port-opening-behind-starlink-purevpn-13.png)


## Ouverture des ports

Créez d'abord vos transferts de ports en allant dans **Firewall** -> **NAT** -> **Port Forward**.
Vous devez utiliser l'interface **PureVPN** et non pas le **WAN** comme ceci : 
![port-opening-behind-starlink-purevpn-14.png](/starlink/nat-behind-starlink/port-opening-behind-starlink-purevpn-14.png)

Maintenant, créez les règles de pare-feu pour autoriser la connexion distante depuis **Firewall** -> **Rules** -> **PureVPN** comme ceci : 

![port-opening-behind-starlink-purevpn-15.png](/starlink/nat-behind-starlink/port-opening-behind-starlink-purevpn-15.png)


Vous pourrez ensuite vérifier l'ouverture de vos ports avec un outil comme https://www.yougetsignal.com/tools/open-ports/ :

![port-opening-behind-starlink-purevpn-16.png](/starlink/nat-behind-starlink/port-opening-behind-starlink-purevpn-16.png)

## Sources
- M4DM4NZ : https://forum.opnsense.org/index.php?topic=4979.0