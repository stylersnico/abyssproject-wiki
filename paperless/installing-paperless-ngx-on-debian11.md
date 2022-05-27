---
title: Installation de Paperless-NGX sur Debian 11
description: Installation de Paperless-NGX sur Debian 11 via Docker et utilisateur dédié
published: true
date: 2022-05-27T06:06:15.347Z
tags: paperless, ged, dms, edm
editor: markdown
dateCreated: 2022-05-27T06:06:15.347Z
---

# Introduction

Le but de cet article est de mettre en place la gestion électronique de documents avec le produit Opensource Paperless-NGX.

> L'accès externe ne sera pas abordé ici. Le système ne disposant pas d'authentification double facteur, je recommande de ne pas le laisser ouvert sur Internet.
{.is-info}


# Installation de Docker

 
Installez les pré-requis :

```bash
apt-get update && apt-get install ca-certificates curl gnupg lsb-release sudo -y
```

Mettez en place le repository de Docker et installez Docker : 
```bash
curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian $(lsb_release -cs) stable" |  tee /etc/apt/sources.list.d/docker.list > /dev/null

apt-get update && apt-get install docker-ce docker-ce-cli containerd.io -y
```

Installez Docker Compose V1 :
```bash
curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
 chmod +x /usr/local/bin/docker-compose
ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose

docker-compose --version
```

# Création de l'utilisateur Paperless

Créez l'utilisateur Linux pour Paperless :
```bash
adduser paperless
```

Ajoutez-le dans les groupes nécessaires et connectez-vous avec : 
```bash
usermod -aG sudo paperless
usermod -aG docker paperless
su - paperless
```


# Installation de Paperless

Lancez le script suivant pour installer PaperLess :
```bash
bash -c "$(curl -L https://raw.githubusercontent.com/paperless-ngx/paperless-ngx/master/install-paperless-ngx.sh)"
```

Plusieurs options seront demandées, personnellement, j'aime bien conserver les media, les data et la database en dehors d'un volume Docker pour une sauvegarde plus simple :

> Activez Apache Tika si vous souhaitez intégrer plus de fichiers que des simples PDF dans votre GED (documents Office par exemple).
{.is-info}


```bash
Summary
=======

Target folder: /home/paperless/paperless-ngx
Consume folder: /home/paperless/paperless-ngx/consume
Media folder: /home/paperless/documents
Data folder: /home/paperless/data
Database (postgres) folder: /home/paperless/database

Port: 8000
Database: postgres
Tika enabled: yes
OCR language: fra
User id: 1001
Group id: 1001
Timezone : Europe/Paris

Paperless username: paperless
Paperless email: paperless@domain.com
```

> A la fin de l'installation, Paperless sera accessible depuis l'adresse suivante : http://IP_ADDRESS:8000/
{.is-success}


## Mise à jour automatique du système

Si vous souhaitez automatiser le processus de mise à jour (ce que vous devriez faire), je vous invite à mettre en place Watchtower : 
- https://wiki.abyssproject.net/fr/docker/watchtower-docker-update