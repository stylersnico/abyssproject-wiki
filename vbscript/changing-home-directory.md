---
title: Changement du répertoire HOME des utilisateurs AD
description: Changement du répertoire HOME des utilisateurs AD
published: true
date: 2023-02-10T08:36:45.994Z
tags: active directory, vbscript
editor: markdown
dateCreated: 2023-02-10T08:36:45.994Z
---

# Introduction

Le but de ce script est de changer les répertoires homes de tous les utilisateurs.

La première variable est l'ancien serveur.
La seconde variable est le nouveau serveur.

# Script


```vbnet
strNewPath 		= "\\srvcompany01\users$\"
strSearchContext 	= "\\chgefs01.company.local\users$\"



Dim rootDSE, domainObject
Set rootDSE = GetObject("LDAP://RootDSE")
domainContainer = rootDSE.Get("defaultNamingContext")
Set domainObject = GetObject("LDAP://" & domainContainer)

Set fs = CreateObject ("Scripting.FileSystemObject")
Set logFile = fs.CreateTextFile (".\ChgHomeDir.log")

const crlf="<BR>"
Set objExplorer = WScript.CreateObject("InternetExplorer.Application")
objExplorer.Navigate "about:blank"   
objExplorer.ToolBar = 0
objExplorer.StatusBar = 0
objExplorer.Width = 500
objExplorer.Height = 300 
objExplorer.Left = 100
objExplorer.Top = 100

Do While (objExplorer.Busy)
   Wscript.Sleep 200
Loop

objExplorer.Visible = 1    
txtOutput=""


If Right(strNewPath, 1) <> "\" then
   strNewPath = strNewPath & "\"
End If
strSearchContext = UCase(strSearchContext)

exportUsers(domainObject)


Set oDomain = Nothing
showText("FINISHED!!")
WScript.Quit



Sub ExportUsers(oObject)
   Dim oUser
   For Each oUser in oObject
      Select Case oUser.Class
         Case "user"
            If oUser.homeDirectory <> "" and InStr(UCase(oUser.homeDirectory), strSearchContext) > 0 then
               showText(oUser.displayName)
               logFile.WriteLine oUser.homeDirectory & "," & strNewPath & oUser.sAMaccountName
               oUser.Put "homeDirectory", strNewPath & oUser.sAMaccountName  
               oUser.SetInfo
            End if
         Case "organizationalUnit" , "container"
            ExportUsers(oUser)
      End select
   Next
End Sub


Sub ShowText(txtInput)
   txtOutput = "Change Home Directory Path" & crlf
   txtOutput = txtOutput & "==========================" & crlf & crlf
   txtOutput = txtOutput & "Search Context: " & strSearchContext & crlf
   txtOutput = txtOutput & "New Home Dir Path: " & strNewPath & crlf & crlf
   txtOutput = txtOutput & "Working on:  " & txtInput
   objExplorer.Document.Body.InnerHTML = txtOutput
End Sub


```