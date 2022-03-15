---
title: Réparation du spooleur d'impression figé sous RDS 2016 / 2019
description: Réparation du spooleur d'impression qui fige complètement sans raison
published: true
date: 2022-03-15T10:17:34.316Z
tags: impression, windows
editor: markdown
dateCreated: 2022-03-15T10:17:34.316Z
---

# Introduction

Le spooleur d'impression se figeait complètement sans raison et sans imprimante installée sous Windows Server 2016 et 2019 avec les rôles Remote Desktop.

Aucun message dans le gestionnaire des tâches.

 
# Isolation du contrôleur de domaine primaire


```powershell
Install-WindowsFeature RSAT-DFS-Mgmt-Con
```
 

 
## Sources

- https://docs.microsoft.com/en-us/answers/questions/451102/server-2016-rds-print-spooler-issues.html