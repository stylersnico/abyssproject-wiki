---
title: Suppression d'un email de toutes les boites en Powershell
description: Suppression d'un email de toutes les boites en Powershell
published: true
date: 2025-05-20T07:51:09.677Z
tags: exchange, exchange online, compliance
editor: markdown
dateCreated: 2025-05-20T07:51:09.677Z
---

# Introduction
Cette procédure permet de supprimer un email présent dans toutes les boites du tenant en cas de réception d'un virus ou d'erreur d'envoi.


# Connexion avec le module Exchange Online


Assurez-vous d'avoir la version 3 du module Exchange Online : 

```powershell
Get-InstalledModule -Name ExchangeOnlineManagement

Version              Name                                Repository           Description
-------              ----                                ----------           -----------
3.2.0                ExchangeOnlineManagement            PSGallery            This is a General Availability (GA) rele…
```

Sinon, mettez à jour et réinstallez le module pour être certain que tout marche, ensuite fermez et réouvrez Powershell : 

```powershell
Update-Module -Name ExchangeOnlineManagement
Install-Module -Name ExchangeOnlineManagement -Force
```

Connectez-vous à Exchange Online avec la commande suivante :
```powershell
Connect-ExchangeOnline -UserPrincipalName admin@tenant.ch
```

Assignez-vous le rôle suivant : 
```powershell
Add-RoleGroupMember "Discovery Management" -member admin@tenant.ch
```

Connectez-vous ensuite à la compliance en plus d'exchange (à la suite de la première commande) : 
```powershell
Connect-IPPSSession -UserPrincipalName admin@tenant.ch
```

Pour finir, assignez-vous le rôle suivant : 
```powershell
Add-eDiscoveryCaseAdmin admin@tenant.ch
```

# Création d'une recherche

> Il faut de la patience pour les recherches. La ligne de commande est ce qui s'actualise le moins vite.
> Le centre en ligne dispose des résultats un peu plus rapidement.
> Enfin, les boites emails seront les premières à jour en cas de suppression, donc en cas d'urgence, basez vous là-dessus.
{.is-warning}

Si vous cherchez un email précis, faites une recherche directement avec le sujet comme ceci : 
```powershell
$Search=New-ComplianceSearch -Name "RemoveMessage_User" -ExchangeLocation all -ContentMatchQuery '(Received:08/31/2023 00:00..08/31/2023 23:59) AND (from:"User.pleux@toto.mail") AND subject:"Re: Validation des notes de frais*"'
```
Vous pouvez également faire une recherche de tous les emails reçus sur une journée ou une plage de date précise uniquement : 
```powershell
$Search=New-ComplianceSearch -Name "RemoveMessage_User" -ExchangeLocation all -ContentMatchQuery '(Received:08/31/2023 00:00..08/31/2023 23:59) AND (from:"User@toto.mail")'
```

N'oubliez pas de lancer votre recherche : 
```powershell
Start-ComplianceSearch “Search User” 
```


# Suivi des résultats

Vous pourrez suivre les résultats avec la commande suivante :
```powershell
Get-ComplianceSearch  "RemoveMessage_User" | FL
```

Si votre filtre est bon, vous verrez les emails apparaitre : 

```powershell
SuccessResults                        : {Location: User.pleux@toto.mail, Item count: 2, Total size: 939535,
                                        Location: 1@toto.mail, Item count: 2, Total size: 545984,
                                        Location: 2@toto.mail, Item count: 1, Total size: 669325,
                                        Location: 3@toto.mail, Item count: 1, Total size: 668918,
                                        Location: 4@toto.mail, Item count: 1, Total size: 668858,
                                        Location: 5@toto.mail, Item count: 1, Total size: 668818,
                                        Location: 6@toto.mail, Item count: 1, Total size: 668809,
```            

> Les emails non filtrés ne seront pas pris en compte par une éventuelle action sur la recherche même s'ils sont affichés.
{.is-info}


# Suppression des emails

Si la recherche est bonne, vous pourrez lancer la suppression **définitive** des emails avec la commande suivante : 
```powershell
New-ComplianceSearchAction -SearchName "RemoveMessage_User" -Purge -PurgeType HardDelete
```

Vous pourrez suivre l'opération avec la commande suivante : 
```powershell
Get-ComplianceSearchAction | fl
```

> Le résultat sera visible dans les boites utilisateurs avant d'être visible et on rappelle que cette commande est destructrice, l'email n'ira pas dans la corbeille !
{.is-danger}
