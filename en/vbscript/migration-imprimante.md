---
title: Migrating printers to another server
description: Migrating printers to another server
published: true
date: 2026-10-09T08:00:00.000Z
tags: active directory, vbscript
editor: markdown
dateCreated: 2026-10-09T08:00:00.000Z
---

# Introduction

The goal of this script is to move printers from one server to another on user workstations after a print server migration.

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

# Usage

Call the script from your usual logon scripts by adding this line:

```batch
cscript //nologo \\chgefs01.domain.local\sysvol\domain.local\scripts\migrate-printers1.vbs
```
