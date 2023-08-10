---
title: Création d'un script batch avec des accents
description: Création d'un script batch avec des accents
published: true
date: 2023-08-10T12:08:16.587Z
tags: msdos, batch, cmd, accents
editor: markdown
dateCreated: 2023-08-10T12:08:16.587Z
---

# Introduction

Batch ne supporte pas par défaut les accents, il faut enregistrer le fichier correctement.


# Explications

Créez votre script dans Wordpad :

```powershell
net use * /delete /y
net use f: \\XXX\filegate\partage1
net use z: \\XXX\filegate\partage12
net use y: \\XXX\filegate\partage3
net use x: \\XXX\filegate\partage4
```

Sauvegardez le fichier en tant qu'autre format : 

![batch-accent-01.png](/windows/batch/accents/batch-accent-01.png)

Sélectionnez le format **ext Document - MS-DOS Format** :

![batch-accent-02.png](/windows/batch/accents/batch-accent-02.png)


Le fichier sera maintenant fonctionnel.