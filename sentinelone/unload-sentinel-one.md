---
title: Désactivation de l'agent Sentinel One
description: Désactivation de l'agent Sentinel One sur un poste pour diagnostic
published: true
date: 2022-03-24T13:06:58.631Z
tags: sentinel one
editor: markdown
dateCreated: 2022-03-24T13:06:58.631Z
---

# Introduction
Le but de ce document est de monter comment désactiver l'agent Sentinel One sur un pc.

# Désactivation de l'agent

> La passphrase est unique pour chaque endpoint, vous pouvez récupérer la passphrase depuis votre console Sentinel One
{.is-info}

![sentinelone_-_management_console.webp](/sentinelone/sentinelone_-_management_console.webp)

Dans un premier temps, rendez-vous dans le dossier d'installation de Sentinel sous Windows (ou lancez directement la commande sous MacOS / Linux) et lancez la commande suivante pour désactiver l'autoprotection de Sentinel One : 
```bash
sentinelctl.exe unprotect -k "endpoint passphrase"
```

Maintenant, déchargez les modules de protection : 
```bash
sentinelctl unload -m -a
```


# Ré-activation de l'agent

Réactivez les modules de protection avec la commande suivante : 
```bash
sentinelctl load -m -a
```
Et, réactivez l'autoprotection de Sentinel One avec la commande suivante : 
```bash
sentinelctl protect
```