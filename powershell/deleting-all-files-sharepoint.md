---
title: Suppression de tous les fichiers d'une bibliothèques Sharepoint
description: Suppression de tous les fichiers d'une bibliothèques Sharepoint en Powershell
published: true
date: 2023-02-10T08:29:26.755Z
tags: powershell, sharepoint
editor: markdown
dateCreated: 2023-02-10T08:29:26.755Z
---

# Introduction

Le but de ce script est de supprimer tous les fichiers présents dans une bibliothèque Sharepoint sans supprimer la bibliothèque elle-même.

# Pré-requis
Vous devez installer le module PNP : 
```powershell
Install-Module PnP.PowerShell
```

# Script

Vous devez d'abord indiquer l'URL du site Sharepoint et le nom de la bibliothèque, ensuite, connectez-vous, listez les fichiers et supprimez tout si cela est correct : 

```powershell
$SiteURL = "https://companysa.sharepoint.com/teams/company"
$ListName = "Bibliothèque de conservation et de préservation"
  
#Connect to PnP Online
Connect-PnPOnline -Url $SiteURL -Interactive

#List Files
Get-PnPList -Identity $ListName | Get-PnPListItem -PageSize 100 | Select-Object id,@{label="Filename";expression={$_.FieldValues.FileLeafRef}}

#Delete all files from the library
Get-PnPList -Identity $ListName | Get-PnPListItem -PageSize 100 -ScriptBlock {
    Param($items) Invoke-PnPQuery } | ForEach-Object { $_.Recycle() 
}
```