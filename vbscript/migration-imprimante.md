---
title: Migration d'imprimantes vers un autre serveur
description: Migration d'imprimantes vers un autre serveur
published: true
date: 2023-08-10T11:55:55.618Z
tags: active directory, vbscript
editor: markdown
dateCreated: 2023-08-10T11:55:55.618Z
---

# Introduction

Le but de ce script est de migrer les imprimantes d'un serveur à l'autre sur des postes utilisateurs après une migration du serveur d'impression.

# Script


```vbnet
strOldServer = "srvdc01"
strNewServer = "chgefs01"

strComputer = "."
Set WSHNetwork = CreateObject("WScript.Network")
Set objWMIService = GetObject("winmgmts:{impersonationLevel=impersonate}!\\" & strComputer & "\root\cimv2")
Set colInstalledPrinters =  objWMIService.ExecQuery("Select * from Win32_Printer")

strOldServer = prepServer(strOldServer)
strNewServer = prepServer(strNewServer)

For Each objPrinter in colInstalledPrinters
   strName = objPrinter.Name
   iPrinterLocation = InStr(UCase(objPrinter.Name),UCase(strOldServer))
   If iPrinterLocation > 0 then
      strPrinter = strNewServer & Right(strName, Len(strName) - Len(strOldServer))
      objPrinter.Delete_
      WSHNetwork.AddWindowsPrinterConnection strPrinter
      If objPrinter.Default = True Then
         WSHNetwork.SetDefaultPrinter strPrinter
      End If
   End If
Next


Function prepServer(strServer)
   If Left(strServer, 2) <> "\\" then
      strServer = "\\" & strServer
   End If
   If Right(strServer, 1) <> "\" then
      strServer = strServer & "\"
   End If
   prepServer = strServer
End Function
```

# Utilisation

Appelez le script depuis les scripts de logon habituels en ajoutant cette ligne : 

```batch
cscript //nologo \\chgefs01.domain.local\sysvol\domain.local\scripts\migrate-printers1.vbs
```