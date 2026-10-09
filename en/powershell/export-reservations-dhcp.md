---
title: Exporting DHCP reservations from Windows Server
description: Exporting DHCP reservations from Windows Server with PowerShell
published: true
date: 2026-10-09T08:00:00.000Z
tags: powershell, dhcp
editor: markdown
dateCreated: 2026-10-09T08:00:00.000Z
---

# Introduction

The goal of this script is to export DHCP reservations to a CSV file.


# Script

```powershell
Get-DHCPServerV4Scope | ForEach {

    Get-DHCPServerv4Lease -ScopeID $_.ScopeID | where {$_.AddressState -like '*Reservation'}

} | Select-Object HostName,ClientID,AddressState | Export-Csv ".\$($env:COMPUTERNAME)-Reservations.csv" -NoTypeInformation
```
