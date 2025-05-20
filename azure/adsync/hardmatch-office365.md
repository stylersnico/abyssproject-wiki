---
title: Réaliser un hard match avec Azure AD
description: Réaliser un hard match avec Azure AD
published: true
date: 2025-05-20T07:44:41.640Z
tags: azure, office 365, hardmatch
editor: markdown
dateCreated: 2025-05-20T07:44:41.640Z
---

# Introduction

Le but ici est de régler les erreurs de synchronisation Azure AD quand un compte existant dans Office 365 ne veut pas s'associer à un compte créé récemment dans AD.


# Réalisation d'un hardmatch

Depuis le powershell du contrôleur AD du client : 

```powershell

# Get GUID for User
$User = Get-ADUser jdupont | select ObjectGUID,UserPrincipalName
$Upn = $User.UserPrincipalName
$Guid = $User.ObjectGUID.Guid
 
# Convert GUID to ImmutableID
$ImmutableId = [System.Convert]::ToBase64String(([GUID]($User.ObjectGUID)).tobytearray())
 
# Connect MsolService
Connect-Msolservice
 
# Set ImmutableID to msoluser
Set-MsolUser -UserPrincipalName $Upn -ImmutableId $ImmutableId
```