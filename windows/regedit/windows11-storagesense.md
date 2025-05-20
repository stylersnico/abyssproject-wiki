---
title: Deployer Windows 11 et StorageSense via registre
description: Deployer Windows 11 et StorageSense via registre
published: true
date: 2025-05-20T07:43:00.526Z
tags: storage sense, windows 11
editor: markdown
dateCreated: 2025-05-20T07:43:00.526Z
---

# Introduction
Ce fichier de registre permet de forcer l'upgrade vers Windows 11 23h2 et de configurer storage sense pour enlever les fichiers temporaires.

> L'upgrade sera forcé sous 7 jours avec accord de l'utilisateur.
> La mise à jour s'installe tous les jours à 12h00.
{.is-info}


# Script

Mettez cela dans un fichier **.reg** et exécutez-le en admin ou déployez-le : 

```powershell
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate]
"TargetReleaseVersion"=dword:00000001
"TargetReleaseVersionInfo"="23H2"
"ProductVersion"="Windows 11"

[HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU]
"AUOptions"=dword:00000004
"ScheduledInstallEveryWeek"=dword:00000001
"ScheduledInstallDay"=dword:00000000
"ScheduledInstallTime"=dword:000000c


[HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows\StorageSense]
"AllowStorageSenseGlobal"=dword:000000001
"ConfigStorageSenseGlobalCadence"=dword:00000007
```