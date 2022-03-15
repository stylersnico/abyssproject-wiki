---
title: Supprimer des utilisateurs Office 365 de la corbeille
description: Suppression de la corbeille avant les 30 jours d'attente
published: true
date: 2022-03-15T10:33:32.010Z
tags: office 365, msonline
editor: markdown
dateCreated: 2022-03-15T10:26:19.648Z
---

# Introduction

Le but de ce document est de supprimer les utilisateurs Office 365 présents dans la corbeille.

> Cette action est définitive. Il ne sera plus possible de récupérer les utilisateurs supprimés.
> {.is-danger}




# Installation du module MSOnline

Installez d'abord le module nécessaire en powershell : 

```powershell
Install-Module msonline
```

Connectez-vous à Office 365 avec la commande suivante : 
```powershell
Connect-MsolService
```

# Suppression des utilisateurs

Vous pouvez lister les utilisateurs présents dans la corbeille avec la commande suivante : 

```powershell
Get-MsolUser -ReturnDeletedUsers
```

Vous pouvez supprimer un utilisateur spécifique avec la commande suivante : 

```powershell
Remove-MsolUser -UserPrincipalName ‘user@contoso.mail.onmicrosoft.com’ -RemoveFromRecycleBin
```
Ou vous pouvez vider l'intégralité de la corbeille avec la commande suivante : 

```powershell
Get-MsolUser -ReturnDeletedUsers | Remove-MsolUser -RemoveFromRecycleBin -Force
```

    
### Sources

Documentation officielle : https://docs.microsoft.com/en-us/powershell/module/msonline/