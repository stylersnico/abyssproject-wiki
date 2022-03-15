---
title: Réparation du spooleur d'impression figé sous RDS 2016 / 2019
description: Réparation du spooleur d'impression qui fige complètement sans raison
published: true
date: 2022-03-15T10:21:38.576Z
tags: impression, windows
editor: markdown
dateCreated: 2022-03-15T10:17:34.316Z
---

# Introduction

Le spooleur d'impression se figeait complètement sans raison et sans imprimante installée sous Windows Server 2016 et 2019 avec les rôles Remote Desktop.

Aucun message dans le gestionnaire d'évènements.

 
# Résolution du problème

Ouvrez Powershell en admin sur le RDS et lancez les commandes suivantes : 

```powershell
net stop spooler
Remove-Item -Path "HKLM:\Software\Microsoft\Windows NT\CurrentVersion\Print\Providers\Client Side Rendering Print Provider*" –Recurse
net start spooler
```
 

 
## Sources

- https://docs.microsoft.com/en-us/answers/questions/451102/server-2016-rds-print-spooler-issues.html