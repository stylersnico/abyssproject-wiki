---
title: Unlocking print spooler with Powershell
description: Unlocking print spooler with Powershell
published: true
date: 2023-02-10T08:10:14.147Z
tags: powershell, print
editor: markdown
dateCreated: 2023-02-10T08:10:14.147Z
---

# Introduction

The goal of this script is to unlock all printer spooler in Powershell.


# Script


```powershell
net stop spooler
Remove-Item C:\Windows\System32\spool\PRINTERS\*.*
net start spooler
```