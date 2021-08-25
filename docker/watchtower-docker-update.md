---
title: Automatisation des updates Docker avec Watchtower
description: Mettez à jour toutes les images lancées avec des docker run / create.
published: true
date: 2021-08-25T13:14:18.175Z
tags: docker, watchtower
editor: markdown
dateCreated: 2021-08-24T16:06:06.326Z
---

# Introduction

Le but de ce document est de mettre à jour automatiquement les images Docker présentes sur votre installation.

Ceci est particulièrement intéressant si vous utilisez Docker en standalone et que vous avez lancer des images avec Docker Run.


# Mise à jour automatique des containers Docker

Une unique commande permets de programmer la mise à jour de vos containers tous les soirs à 23h par exemple : 

```bash
docker run -d \
    --name watchtower \
    -v /var/run/docker.sock:/var/run/docker.sock \
    --restart always \
    containrrr/watchtower \
    --schedule "0 23 * * *" \
    --cleanup
```

Dans le détail, voici les arguments les plus intéressants :

- Ici on redémarre toujours le conteneur Watchtower, peut importe s'il recontre une erreur lors de l'exécution qui le planterais.
```bash
    --restart always \
```

- Ici on planifie les mises à jours des conteneurs Docker chaque soir à 23h00.
```bash
    --schedule "0 23 * * *" \
```

- Ici on supprime automatiquement les anciennes versions d'images Docker qui ne sont plus nécessaires afin de garder de l'espace disque.
```bash
    --cleanup
``` 
    
### Sources

Documentation officielle : https://containrrr.dev/watchtower/arguments/