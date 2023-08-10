---
title: Export des réservations DHCP de WIndows Serveur
description: Export des réservations DHCP de Wnndows Serveur en Powershell
published: true
date: 2023-08-10T12:00:43.919Z
tags: powershell, dhcp
editor: markdown
dateCreated: 2023-08-10T12:00:43.919Z
---

# Introduction

Le but de ce script est de récupérer les réservations DHCP au format CSV.


# Script

```powershell
Get-DHCPServerV4Scope | ForEach {

    Get-DHCPServerv4Lease -ScopeID $_.ScopeID | where {$_.AddressState -like '*Reservation'}

} | Select-Object HostName,ClientID,AddressState | Export-Csv ".\$($env:COMPUTERNAME)-Reservations.csv" -NoTypeInformation
```