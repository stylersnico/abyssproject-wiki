---
title: Configuration de Storage Sense en Powershell
description: Configuration de Storage Sense en Powershell
published: true
date: 2023-02-10T07:57:28.580Z
tags: powershell, storage sense, storage, sense
editor: markdown
dateCreated: 2023-02-10T07:57:28.580Z
---

# Introduction

Le but de ce script est de configurer l'assistant de stockage (Stockage Sense) en powershell afin de libérer automatiquement les corbeilles et le dossier téléchargement.


# Script

Les valeurs dans ce script efface les documents vieux de plus de 14 jours dans la corbeille et le dossier téléchargement chaque jour : 

```powershell
$storagePolicy = "HKCU:\Software\Microsoft\Windows\CurrentVersion\StorageSense\Parameters\StoragePolicy"

# Enabling Storange Sense: 
# 0x00000000 = Disabled
# 0x00000001 = Enabled
Set-ItemProperty $storagePolicy -Name "01" -Value 0x00000001; 

# Delete temporary files that my apps aren’t using: 
# 0x00000000 = Disabled
# 0x00000001 = Enabled
Set-ItemProperty $storagePolicy -Name "04" -Value 0x00000001; 

# Delete files in my recycle bin if they have been there for over
# 0x00000000 = Never
# 0x00000001 = 1 Day
# Please note: 0x00000001 needs to be set if you want to utilise more than 1 Day with value "256"
Set-ItemProperty $storagePolicy -Name "08" -Value 0x00000001; 

# Delete files in my recycle bin if they have been there for over
# 0x00000000 = Never
# 0x00000001 = 1 Day
# 0x0000000E = 14 Days
# 0x0000001E = 30 Days
# 0x0000003C = 60 Days
# Please Note: Value "04" needs to be set as a REG_DWORD with a data value of 0x00000001 in order for this to work for 1 Day or More, likewise for Never being 0x00000000
Set-ItemProperty $storagePolicy -Name "256" -Value 0x0000000E; 

# Delete files in my Downloads folder if they have been there for over
# 0x00000000 = Never
# 0x00000001 = 1 Day
# Please note: 0x00000001 needs to be set if you want to utilise more than 1 Day with value "512"
Set-ItemProperty $storagePolicy -Name "32" -Value 0x00000001; 

# Delete files in my Downloads folder if they have been there for over
# 0x00000000 = Never
# 0x00000001 = 1 Day
# 0x0000000E = 14 Days
# 0x0000001E = 30 Days
# 0x0000003C = 60 Days
# Please Note: Value "32" needs to be set as a REG_DWORD with a data value of 0x00000001 in order for this to work for 1 Day or More, likewise for Never being 0x00000000
Set-ItemProperty $storagePolicy -Name "512" -Value 0x0000000E; 

# Run Storage Sense
# 0x00000001 = Everyday
# 0x00000007 = Every 7 Days
# 0x0000001E = Every Month
# 0x00000000 = During Low Disk Space
Set-ItemProperty $storagePolicy -Name "2048" -Value 0x00000001; 
```

## Source

- https://gist.github.com/christopher-talke/146513b852cef21a917cb1d9ecb25d9a