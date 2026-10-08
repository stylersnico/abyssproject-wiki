---
title: 3 - Virtualisation Hyper-V
description: Virtualisation Hyper-V
published: true
date: 2026-10-08T14:04:28.562Z
tags: infra
editor: markdown
dateCreated: 2026-10-08T14:04:28.562Z
---

# Virtualisation Hyper-V
Les services tournent sur un seul hôte **Hyper-V**. Les machines virtuelles sont numérotées (110 à 118) pour les retrouver facilement dans la console et dans les sauvegardes.

# Hôte

| Élément | Valeur |
| --- | --- |
| Matériel | Supermicro Super Server |
| Processeur | Intel Xeon D-1518, 4 cœurs / 8 threads |
| Mémoire | 64 Go |
| Système | Windows Server 2025 Datacenter, groupe de travail (hors domaine) |
| Réseau | un lien 10 GbE (Intel X552) portant un commutateur virtuel externe, partagé avec l’hôte |
| Stockage | volume système de 931 Go (système et machines virtuelles), volume de données de 13 039 Go |


# Machines virtuelles

Toutes les machines sont en génération 2, avec mémoire statique, et démarrent automatiquement avec l’hôte.

| Machine | Système | vCPU | Mémoire | Disque | Rôle |
| --- | --- | --- | --- | --- | --- |
| 110-Webhost | Debian | 6 | 8 Go | 350 Go (fixe) | sites et applications web : NGINX, PHP 8.4, MariaDB, Docker |
| 111-HAOS | Home Assistant OS | 4 | 4 Go | 32 Go | domotique |
| 112-Reverse | FreeBSD | 4 | 4 Go | 20 Go (dynamique) | reverse proxy NGINX, CrowdSec |
| 113-Ansible | Debian | 4 | 1 Go | 40 Go (dynamique) | Ansible et Semaphore |
| 114-Paperless | Debian | 4 | 2 Go | 40 Go (dynamique) | gestion documentaire Paperless |
| 115-CheckMK | Non documenté | 6 | 8 Go | 50 Go (fixe) | supervision Checkmk |
| 116-Wazuh | Ubuntu | 6 | 6 Go | 100 Go (dynamique) | SIEM Wazuh |
| 117-Media | Debian | 6 | 12 Go | 8 To (dynamique, volume de données) | médiathèque Jellyfin |
| 118-Passbolt | Non documenté | 4 | 2 Go | 30 Go (fixe) | gestionnaire de mots de passe Passbolt |

Au total, 44 vCPU et 47 Go de mémoire sont alloués sur 8 threads et 64 Go.
