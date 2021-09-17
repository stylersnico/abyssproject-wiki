---
title: Mise à jour des emails Exchange en masse
description: Le tout en powershell ...
published: true
date: 2021-09-17T07:49:28.395Z
tags: exchange, powershell, email
editor: markdown
dateCreated: 2021-09-17T07:46:26.095Z
---

# Introduction

Le but de ce script est de renommer les adresses SMTP dans Exchange afin d'uniformiser le parc d'utilisateurs.


# Script

Dans un IDE powershell:

```powershell
Add-PSSnapin Microsoft.Exchange.Management.PowerShell.SnapIn
$MailboxList = Get-Mailbox  -ResultSize unlimited
$MailboxList | % {
 
$LoweredList = @()
$RenamedList = @()
 
foreach ($Address in $_.EmailAddresses){
if ($Address.prefixstring -eq "SMTP"){
$RenamedList += $Address.smtpaddress + "TempRename"
$LoweredList += $Address.smtpaddress.ToLower()
}
}
Set-mailbox $_ -emailaddresses $RenamedList -EmailAddressPolicyEnabled $false
Set-mailbox $_ -emailaddresses $LoweredList
 
#Without this line the "Reply To" Address could be lost on recipients with more than one proxy address:
Set-mailbox $_ -PrimarySmtpAddress $_.PrimarySmtpAddress
}
```


    
## Sources

Converting SMTP Proxy Addresses to Lowercase by Mike Crowley: https://mikecrowley.us/2012/05/14/converting-smtp-proxy-addresses-to-lowercase/