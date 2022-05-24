---
title: Backup d'un serveur Debian avec Restic
description: Sauvegarde complète d'un serveur Debian avec Restic
published: true
date: 2022-05-24T12:15:00.009Z
tags: debian, backup, restic
editor: markdown
dateCreated: 2022-05-24T12:09:38.197Z
---

# Introduction

Le but de cet article est de réaliser la sauvegarde partielle ou complète d'un serveur Debian avec Restic.

> Dans ce cas, une [Storage Box](https://www.hetzner.com/storage/storage-box) d'Hetzner sera utilisée avec connexion via SFTP et authentification par clé privée.
{.is-info}


# Installation

 
Installez Restic :

```bash
apt update
apt install restic
```

# Configuration de l'emplacement de sauvegarde

> Si vous utilisez du SFTP vous devez obligatoirement faire l'authentification via échange de clé SSH, [voici le guide pour le faire avec une Storage Box](https://docs.hetzner.com/robot/storage-box/backup-space-ssh-keys/).
{.is-warning}


Utilisez la commande suivante pour initialiser le repository :
>Vous devrez mettre deux fois le mot de passe qui permettra de chiffrer les données en AES256, notez bien ce mot de passe !
{.is-danger}
```bash
restic -r sftp:uXXXXXX@uXXXXXX.your-storagebox.de:/srv/restic-repo init
```

Voici le retour que vous devriez avoir : 
```bash
restic -r sftp:uXXXXXX@uXXXXXX.your-storagebox.de:/srv/restic-repo init
enter password for new repository:
enter password again:
created restic repository 57701a571c at sftp:uXXXXXX@XXXXXX.your-storagebox.de:/srv/restic-repo

Please note that knowledge of your password is required to access
the repository. Losing your password means that your data is
irrecoverably lost.
```


# Sauvegarde du serveur

Pour lancer une sauvegarde complète du serveur, vous pouvez utiliser la commande suivante :

> Dans l'exemple, j'ajoute également une exclusion pour un fichier **/var/swap.img**, vous pouvez exclure des fichiers ou des dossiers de la sauvegarde avec cette méthode
{.is-info}
```bash
restic -v -r sftp:uXXXXXX@uXXXXXX.your-storagebox.de:/srv/restic-repo backup --exclude /var/swap.img --one-file-system /
```

Si vous souhaitez sauvegarder un dossier en particulier, vous pouvez simplifier la commande :

```bash
restic -v -r sftp:uXXXXXX@uXXXXXX.your-storagebox.de:/srv/restic-repo backup /var/www/
```

# Automatisation

Si vous souhaitez automatiser le processus (ce que vous devriez faire), ajoutez ceci à la fin de votre commande afin d'obtenir des fichiers de logs beaucoup plus lisibles :
```bash
 | cat
```

Vous devez aussi indiquer le mot de passe de votre repository Restic dans une variable comme ceci par exemple :
```bash
export RESTIC_PASSWORD=repo_password
```

Par exemple : 

```bash
export RESTIC_PASSWORD=repo_password
restic -v -r sftp:uXXXXXX@uXXXXXX.your-storagebox.de:/srv/restic-repo backup --exclude /var/swap.img --one-file-system /  | cat
```