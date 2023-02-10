---
title: Nettoyage Tape avant backup Veeam
description: Nettoyage de la Tape avant backup Veeam
published: true
date: 2023-02-10T08:23:06.426Z
tags: backup, powershell, veeam, tape
editor: markdown
dateCreated: 2023-02-10T08:23:06.426Z
---

# Introduction

Le but de ce script est de nettoyer la tape avant un backup Veeam afin de se libérer des anciennes données.


# Récupération du nom du lecteur

Ouvrez Veeam et notez le nom du lecteur de tape : 

![veeam-tape-erase-before-backup-01.png](/scripts/powershell/veeam-tape-erase-before-backup-01.png)

# Script

Le script inventorie la tape présente dans le lecteur et fait un effacement rapide des données présentes sur la tape : 

```powershell
Start-VBRTapeInventory -library "HP Ultrium 7-SCSI"
$tape = Get-VBRTapeMedium –library "HP Ultrium 7-SCSI"
Erase-VBRTapeMedium –Medium $tape
sleep 120
```

# Utilisation

Enregistrez ce script dans un fichier Powershell.
Ouvrez votre job to tape et allez dans les options avancées du Job : 

![veeam-tape-erase-before-backup-02.png](/scripts/powershell/veeam-tape-erase-before-backup-02.png)

Configurez votre script comme ceci dans le job : 

![veeam-tape-erase-before-backup-03.png](/scripts/powershell/veeam-tape-erase-before-backup-03.png)