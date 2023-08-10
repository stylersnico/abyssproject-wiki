---
title: Nettoyage des Store de Desktop Central
description: Nettoyage des Store de Desktop Central via script VBS
published: true
date: 2023-08-10T11:57:11.332Z
tags: desktop central, vbs
editor: markdown
dateCreated: 2023-08-10T11:57:11.332Z
---

# Introduction

Le but des deux scripts suivants est de nettoyer les anciens fichiers de mise à jour qui ne sont pas supprimés automatiquement


# Script VBS pour le serveur central

```vbnet
'Manage Engine Desktopcentral.
'Script to delete old patch files from Central Server store
'=======================================================================================
'Patches will be deleted if it is downloaded before the days mentioned below.
strDays = 60

'========================================================================================

Set fso = CreateObject("Scripting.FileSystemObject")
patchstoreRegKey = "D:\DesktopCentralMSP_Server\webapps\DesktopCentral\store"
Set f = fso.GetFolder(patchstoreRegKey)
Set fc = f.Files

For Each f1 in fc
      If DateDiff("d", f1.DateLastModified, Date) > strDays Then
            fso.DeleteFile(f1)
      End If
Next
```

# Script VBS pour les serveurs de distribution

```vbnet
'Manage Engine Desktopcentral.
'Script to delete old patch files from Distribution Server store
'=======================================================================================


'Patches will be deleted if it is downloaded before the days mentioned below.
strDays = 60

'========================================================================================

Set WshShell = WScript.CreateObject("WScript.Shell")
Set fso = CreateObject("Scripting.FileSystemObject")


checkOSArch = WshShell.RegRead("HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Session Manager\Environment\PROCESSOR_ARCHITECTURE")

if Err Then
	Err.Clear
	'WScript.Echo "The OS Architecture is unable to find ,so it was assumed to be 32 bit"
	regkey = "HKEY_LOCAL_MACHINE\SOFTWARE\AdventNet\DesktopCentral\DCDistributionServer\"
else
	if checkOSArch = "x86" Then
		'Wscript.Echo "The OS Architecture is 32 bit"
		regkey = "HKEY_LOCAL_MACHINE\SOFTWARE\AdventNet\DesktopCentral\DCDistributionServer\"
	else
		'Wscript.Echo "The OS Architecture is 64 bit"
		regkey = "HKEY_LOCAL_MACHINE\SOFTWARE\Wow6432Node\AdventNet\DesktopCentral\DCDistributionServer\"
	End IF
End If

currentInstallPath = WshShell.regread(regkey&"DCAgentInstallDir")
patchstoreRegKey = currentInstallPath &"replication\store\"

Set f = fso.GetFolder(patchstoreRegKey)
Set fc = f.Files

For Each f1 in fc
      If DateDiff("d", f1.DateLastModified, Date) > strDays Then
            fso.DeleteFile(f1)
      End If
Next


```