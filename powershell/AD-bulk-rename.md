---
title: Mise à jour des utilisateurs AD en masse
description: En powershell, pour uniformiser les majuscules et minuscules.
published: true
date: 2021-09-17T08:00:17.643Z
tags: powershell, ad, active directory
editor: markdown
dateCreated: 2021-09-17T08:00:17.643Z
---

# Introduction

Le but de ce script est de renommer en masse les utilisateurs dans Active Directory afin d'uniformiser les majuscules et miniscules.
La première lettre du prénom et du nom de famille seront en majuscules.


# Script

Dans un IDE powershell:

```powershell
ForEach ($User in (Get-ADUser -Filter * -SearchBase "OU=Administration,OU=COMPANY USER and Group,DC=COMPANY,DC=ch"))
{   $FirstName = $User.GivenName[0].ToString().ToUpper() + $User.GivenName.SubString(1).ToLower()
    $LastName = $User.Surname[0].ToString().ToUpper() + $User.Surname.SubString(1).ToLower()
    Write-Host "First Name: $FirstName`nLast Name: $LastName"
    Set-ADUser $User.SamAccountName -GivenName $FirstName -Surname $LastName -DisplayName "$FirstName $LastName"
    Rename-ADObject -Identity $User -NewName "$FirstName $LastName"
}
```


    
## Sources
Une discussion que je ne retrouve pas sur stack overflow, si vous retrouvez, merci de commenter.