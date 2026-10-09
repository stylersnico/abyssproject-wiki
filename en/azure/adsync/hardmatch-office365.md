---
title: Performing a hard match with Azure AD
description: Performing a hard match with Azure AD
published: true
date: 2026-10-09T08:00:00.000Z
tags: azure, office 365, hardmatch
editor: markdown
dateCreated: 2026-10-09T08:00:00.000Z
---

# Introduction

The goal here is to fix Azure AD synchronization errors when an existing Office 365 account refuses to match a recently created AD account.


# Performing a hard match

From PowerShell on the customer's AD domain controller:

```powershell

# Get GUID for User
$User = Get-ADUser jdupont | select ObjectGUID,UserPrincipalName
$Upn = $User.UserPrincipalName
$Guid = $User.ObjectGUID.Guid
 
# Convert GUID to ImmutableID
$ImmutableId = [System.Convert]::ToBase64String(([GUID]($User.ObjectGUID)).tobytearray())
 
# Connect MsolService
Connect-Msolservice
 
# Set ImmutableID to msoluser
Set-MsolUser -UserPrincipalName $Upn -ImmutableId $ImmutableId
```
