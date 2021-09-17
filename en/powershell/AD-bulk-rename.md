---
title: Bulk update of AD users
description: With powershell, to standardize uppercase and lowercase
published: true
date: 2021-09-17T08:00:54.353Z
tags: powershell, ad, active directory
editor: markdown
dateCreated: 2021-09-17T08:00:54.353Z
---

# Introduction

The goal of this script is to standardize all the users in Active Directory so you can have the same uppercase and lowercase everywere.

The first letter of firstname and last name will be uppercase, everything else in lowercase.

# Script

In a powershell IDE:

```powershell
ForEach ($User in (Get-ADUser -Filter * -SearchBase "OU=Administration,OU=COMPANY USER and Group,DC=COMPANY,DC=ch"))
{   $FirstName = $User.GivenName[0].ToString().ToUpper() + $User.GivenName.SubString(1).ToLower()
    $LastName = $User.Surname[0].ToString().ToUpper() + $User.Surname.SubString(1).ToLower()
    Write-Host "First Name: $FirstName`nLast Name: $LastName"
    Set-ADUser $User.SamAccountName -GivenName $FirstName -Surname $LastName -DisplayName "$FirstName $LastName"
    Rename-ADObject -Identity $User -NewName "$FirstName $LastName"
}
```


    
## Sources
Some discussion on stackoverflow for the first part of the script, can't remember, please comment.