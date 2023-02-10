---
title: Déblocage d'une file d'impression en Powershell
description: Déblocage d'une file d'impression en Powershell
published: true
date: 2023-02-10T08:08:54.365Z
tags: powershell, print
editor: markdown
dateCreated: 2023-02-10T08:08:54.365Z
---

# Introduction

Le but de ce script est de débloquer une file d'impression en supprimant automatique tous les fichiers bloquants.


# Script


```powershell
net stop spooler
Remove-Item C:\Windows\System32\spool\PRINTERS\*.*
net start spooler
```