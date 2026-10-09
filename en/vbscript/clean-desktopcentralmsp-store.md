---
title: Cleaning Desktop Central stores
description: Cleaning Desktop Central stores with a VBS script
published: true
date: 2026-10-09T08:00:00.000Z
tags: desktop central, vbs
editor: markdown
dateCreated: 2026-10-09T08:00:00.000Z
---

# Introduction

The goal of the following two scripts is to clean up old patch files that are not deleted automatically.


# VBS script for the central server

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

# VBS script for distribution servers

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
