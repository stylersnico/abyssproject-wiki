---
title: Certificats Let's Encrypt avec Acme.SH pour Nginx
description: Obtenez des certificats ECDSA chez Let's Encrypt avec Acme.sh
published: true
date: 2021-08-12T16:52:37.947Z
tags: nginx, let's encrypt, acme.sh, ssl, debian
editor: markdown
dateCreated: 2021-08-11T17:36:42.808Z
---

# Introduction

Le but de cet article est d'utiliser l'utilitaire acme.sh afin de générer des certificats ECDSA fournis par l'autorité Let's Encrypt et mis en place dans NGINX.

# Installation

 
Installez acme.sh :

```bash
curl https://get.acme.sh | sh -s email=test@tap.ovh
cd /root/.acme.sh/
chmod +x acme.sh
sh acme.sh --set-default-ca  --server  letsencrypt
```

# Émission d'un certificat
Exécutez la commande suivante en indiquant votre vhost pour demander un certificat :

```bash
sh acme.sh  --issue  -d website.tap.ovh  --nginx /etc/nginx/sites-enabled/wordpress.vhost --keylength ec-384
```

Si l’opération réussie, vous devrez juste configurer le certificat ECDSA dans votre vhost nginx :
```bash
[Wed 11 Aug 2021 08:21:06 PM CEST] Your cert is in: /root/.acme.sh/website.tap.ovh_ecc/website.tap.ovh.cer
[Wed 11 Aug 2021 08:21:06 PM CEST] Your cert key is in: /root/.acme.sh/website.tap.ovh_ecc/website.tap.ovh.key
[Wed 11 Aug 2021 08:21:06 PM CEST] The intermediate CA cert is in: /root/.acme.sh/website.tap.ovh_ecc/ca.cer
[Wed 11 Aug 2021 08:21:06 PM CEST] And the full chain certs is there: /root/.acme.sh/website.tap.ovh_ecc/fullchain.cer
```

# Renouvellement automatique

Tous les certificats sont automatiquement renouvelés tous les 60 jours.

