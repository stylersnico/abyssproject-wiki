---
title: Deploying Windows 11 and Storage Sense through the registry
description: Deploying Windows 11 and Storage Sense through the registry
published: true
date: 2026-10-09T08:00:00.000Z
tags: storage sense, windows 11
editor: markdown
dateCreated: 2026-10-09T08:00:00.000Z
---

# Introduction
This registry file forces the upgrade to Windows 11 23H2 and configures Storage Sense to remove temporary files.

> The upgrade will be forced within 7 days, with the user's consent.
> The update installs every day at 12:00.
{.is-info}


# Script

Put this in a **.reg** file and run it as administrator, or deploy it:

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
