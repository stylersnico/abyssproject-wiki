---
title: Deleting all files on a Sharepoint library
description: Deleting all files on a Sharepoint library in PowerShell
published: true
date: 2023-02-10T08:31:45.907Z
tags: powershell, sharepoint
editor: markdown
dateCreated: 2023-02-10T08:31:45.907Z
---

# Introduction

The goal of this script is to delete all files inside a Sharepoint Library without deleting the library itself.

# Prerequisites
You need to install the PNP module first : 
```powershell
Install-Module PnP.PowerShell
```

# Script

You need to configure the URL of your sharepoint and the name of the library, then connect and list all files.
If it's correct, you can delete everything.

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