---
title: Certificats ECDSA Wildcard avec Let's Encrypt et Infomaniak sous Debian
description: Pbtenez des certificats ECDSA Wildcard avec Let's Encrypt et Infomaniak sous Debian
published: false
date: 2026-01-12T09:51:51.177Z
tags: let's encrypt, debian, infomaniak
editor: markdown
dateCreated: 2026-01-12T09:51:51.177Z
---

# Introduction

Le but de cet article est de vous expliquer comment générer des certificats ECDSA Wildcard (donc ***.abyssproject.net** par exemple) avec Let's Encrypt et l'API Infomaniak pour la gestion automatique des DNS.
Le tout est fait avec Debian 13 dans cet exemple.

# Installation des prérequis

 
Installez les paquets suivants sous Debian pour avoir Certbot et son plugin pour aller interargir avec les DNS Infomaniak :

```bash
apt install certbot python3 python3-certbot-dns-infomaniak
```

# Génération de la clé d'api Infomaniak et du fichier de configuration

Allez dans votre manager Infomaniak, dans votre profil et dans la section **Développeur** :
![letsencrypt-infomaniak-dns01.png](/debian/webserver/ecdsa-letsencrypt/letsencrypt-infomaniak-dns01.png)

Ensuite, allez dans **Tokens API** et créez un token comme ceci avec le scope d'accès **domaines** : 
![letsencrypt-infomaniak-dns02.png](/debian/webserver/ecdsa-letsencrypt/letsencrypt-infomaniak-dns02.png)
![letsencrypt-infomaniak-dns03.png](/debian/webserver/ecdsa-letsencrypt/letsencrypt-infomaniak-dns03.png)


Maintenant, créez un fichier de configuration pour Cerbot avec la clé d'API que vous venez de récupérer : 
```bash
echo "dns_infomaniak_token=TOKEN" > /root/infomaniak-credentials.ini
chmod 600 /root/infomaniak-credentials.ini
```

# Émission d'un certificat
Exécutez la commande suivante en indiquant votre vhost pour demander un certificat :

```bash
certbot certonly \
  --authenticator dns-infomaniak \
  --dns-infomaniak-credentials /root/infomaniak-credentials.ini \
  --server https://acme-v02.api.letsencrypt.org/directory \
  --agree-tos \
  --key-type ecdsa \
  -d abyssproject.net -d '*.abyssproject.net'
```

Si l’opération réussit, vous devrez juste configurer le certificat ECDSA dans votre vhost nginx, par exemple :
```bash
        ssl_certificate /etc/letsencrypt/live/abyssproject.net/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/abyssproject.net/privkey.pem;

```

# Test du renouvellement automatique 

Tous les certificats sont automatiquement renouvelés avant expiration.
```bash
certbot renew --dry-run
```
```
Saving debug log to /var/log/letsencrypt/letsencrypt.log

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Processing /etc/letsencrypt/renewal/abyssproject.net.conf
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Simulating renewal of an existing certificate for *.abyssproject.net and abyssproject.net
Waiting 120 seconds for DNS changes to propagate

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Processing /etc/letsencrypt/renewal/nicolas-simond.ch.conf
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Simulating renewal of an existing certificate for nicolas-simond.ch and *.nicolas-simond.ch
Waiting 120 seconds for DNS changes to propagate

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Congratulations, all simulated renewals succeeded:
  /etc/letsencrypt/live/abyssproject.net/fullchain.pem (success)
  /etc/letsencrypt/live/nicolas-simond.ch/fullchain.pem (success)
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
```
