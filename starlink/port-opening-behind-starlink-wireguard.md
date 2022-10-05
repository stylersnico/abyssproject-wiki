---
title: Ouverture de ports sur une connexion Starlink via un tunnel Wireguard
description: Ouverture de ports sur une connexion Starlink via un tunnel Wireguard sur un VPS
published: true
date: 2022-10-05T13:42:27.765Z
tags: opnsense, starlink, nat, wireguard
editor: markdown
dateCreated: 2022-10-05T13:41:32.292Z
---

# Introduction
Le but de cette procédure est de pouvoir ouvrir des ports en entrée sur une connexion Starlink via un serveur Wireguard externe.

> Dans cet exemple, un vps chez OVH sera utilisé, mais il n'y aucune affiliation avec eux.
{.is-info}

Vous devez également avoir un firewall comme OPNSense sur votre connexion Starlink qui servira déjà de firewall évidemment et ensuite permettra de faire du nat uniquement sur les ports souhaités.

Exemple de configuration d'un firewall OPNSense avec Starlink : https://wiki.abyssproject.net/fr/starlink/connecting-starlink-opnsense


# Configuration du serveur Wireguard

Dans cet exemple, nous utilisons un VPS Starter chez OVH sous Debian 11 : https://www.ovh.com/fr/order/vps/?v=3#/vps/

Installez Wireguard avec le script de Nyr qui est disponible ici : 

```bash
wget https://git.io/wireguard -O wireguard-install.sh && bash wireguard-install.sh
```

A la fin, le fichier conf sera créé avec toutes les informations nécessaires :
```bash
root@vps:~# cat opnsense.conf
[Interface]
Address = 10.7.0.2/24
DNS = 1.1.1.1, 1.0.0.1
PrivateKey = XXX
[Peer]
PublicKey = XXX
PresharedKey = XXX
AllowedIPs = 0.0.0.0/0, ::/0
Endpoint = XXX:51820
PersistentKeepalive = 25
root@vps:~#
```

# Configuration du firewall OPNSense

## Configuration du client WireGuard

Allez d'abord dans **System** -> **Firmware** -> **Plugins** et installez le plugin **os-wireguard** comme ceci :

![opnsense-wireguard-nat-01.png](/starlink/nat-behind-starlink/wireguard/opnsense-wireguard-nat-01.png)


Maintenant, allez dans **VPN** -> **WireGuard** -> **Endpoint** et ajoutez un endpoint : 

![opnsense-wireguard-nat-02.png](/starlink/nat-behind-starlink/wireguard/opnsense-wireguard-nat-02.png)

Ajoutez le endpoint  comme ceci avec les informations de la section **[Peer]** du fichier de configuration de la section précédente :

![opnsense-wireguard-nat-03.png](/starlink/nat-behind-starlink/wireguard/opnsense-wireguard-nat-03.png)

Allez maintenant dans l'onglet **Local** et cliquez sur le **+** :

![opnsense-wireguard-nat-04.png](/starlink/nat-behind-starlink/wireguard/opnsense-wireguard-nat-04.png)

ajoutez l'interface locale comme ceci avec les informations de la section **[Interface]** du fichier de configuration de la section précédente, pensez-bien à désactiver les routes : 

![opnsense-wireguard-nat-05.png](/starlink/nat-behind-starlink/wireguard/opnsense-wireguard-nat-05.png)

Activez maintenant le service WireGuard depuis l'interface **General** :
![opnsense-wireguard-nat-06.png](/starlink/nat-behind-starlink/wireguard/opnsense-wireguard-nat-06.png)

Attendez quelques secondes et allez dans **List Configuration** pour vérifier que des données sont échangées :
![opnsense-wireguard-nat-07.png](/starlink/nat-behind-starlink/wireguard/opnsense-wireguard-nat-07.png)


## Configuration de l'interface WireGuard

Allez dans **Interfaces** -> **Assignments** et assignez l'interface **wg1** sur un port optionnel : 

![opnsense-wireguard-nat-08.png](/starlink/nat-behind-starlink/wireguard/opnsense-wireguard-nat-08.png)

Allez dans **Interfaces** -> **Wireguard/opt1** -> Activez l'interface sans aucune configuration IP :

![opnsense-wireguard-nat-09.png](/starlink/nat-behind-starlink/wireguard/opnsense-wireguard-nat-09.png)


## Configuration de la passerelle

Allez dans **System** -> **Gateways** -> **Single** et ajoutez une passerelle :

![opnsense-wireguard-nat-10.png](/starlink/nat-behind-starlink/wireguard/opnsense-wireguard-nat-10.png)


Configurez la passerelle comme ceci, il faut bien utiliser l'interface **Wireguard** créée dans le chapitre précédent et indiquer la passerelle en **Far Gateway** : 

![opnsense-wireguard-nat-11.png](/starlink/nat-behind-starlink/wireguard/opnsense-wireguard-nat-11.png)


## Configuration des règles Firewall

Pour autoriser la communication dans le tunnel, créez la règle suivante : 

![opnsense-wireguard-nat-12.png](/starlink/nat-behind-starlink/wireguard/opnsense-wireguard-nat-12.png)


## Configuration de la sortie des clients via le firewall

Créez un alias WireguardClients qui contiendra les hôtes qui doivent sortir sur Internet via le tunnel Wireguard :

![opnsense-wireguard-nat-13.png](/starlink/nat-behind-starlink/wireguard/opnsense-wireguard-nat-13.png)

Créez ensuite les deux règles LAN suivante, la 1ère permettra aux clients dans le groupe de sortir via le tunnel et la 2ème fera sortir tout le reste via le WAN :

![opnsense-wireguard-nat-14.png](/starlink/nat-behind-starlink/wireguard/opnsense-wireguard-nat-14.png)


## Configuration du nat sortant

Allez dans **Firewall** -> **NAT** -> **Outbound** et passez le firewall en **Hybrid outbound NAT rule Generation**.

Créez ensuite les règles manuelles suivantes : 

![opnsense-wireguard-nat-15.png](/starlink/nat-behind-starlink/wireguard/opnsense-wireguard-nat-15.png)


## Ouverture de ports

Allez dans **Firewall** -> **NAT** -> **Port Forward** et créez les règles dont vous avez besoin selon le format suivant : 
![opnsense-wireguard-nat-16.png](/starlink/nat-behind-starlink/wireguard/opnsense-wireguard-nat-16.png)

Allez dans **Firewall** -> **Rules** -> **Wireguard**  et créez les règles correspondantes aux Port Forward que vous avez faits :
![opnsense-wireguard-nat-17.png](/starlink/nat-behind-starlink/wireguard/opnsense-wireguard-nat-17.png)


Vous pourrez ensuite vérifier l'ouverture de vos ports avec un outil comme https://www.yougetsignal.com/tools/open-ports/ :

![port-opening-behind-starlink-purevpn-16.png](/starlink/nat-behind-starlink/port-opening-behind-starlink-purevpn-16.png)
