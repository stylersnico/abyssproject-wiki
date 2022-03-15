---
title: Delete Office 365 users from recycle bin
description: Delete Office 365 users from recycle bin before the 30 days delay
published: true
date: 2022-03-15T10:38:40.493Z
tags: office 365, msonline
editor: markdown
dateCreated: 2022-03-15T10:38:40.493Z
---

# Introduction

The goal is to delete all the Office 365 users in the online recycle bin.

> This action is irreversible. Please be sure of what you are doing.
> {.is-danger}




# Installing the MSonline module

Install the module in powershell: 

```powershell
Install-Module msonline
```

Connect to Office 365 service with the command: 
```powershell
Connect-MsolService
```

# Deleting users

You can list all the users in the recycle bin with the following command: 

```powershell
Get-MsolUser -ReturnDeletedUsers
```

You can delete a specific user with the following command: 

```powershell
Remove-MsolUser -UserPrincipalName ‘user@contoso.mail.onmicrosoft.com’ -RemoveFromRecycleBin
```
Or you can empty the whole recycle bin with this command: 

```powershell
Get-MsolUser -ReturnDeletedUsers | Remove-MsolUser -RemoveFromRecycleBin -Force
```

    
### Sources

Official documentation : https://docs.microsoft.com/en-us/powershell/module/msonline/