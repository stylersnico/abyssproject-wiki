---
title: Bulk update of email addresses in Exchange
description: In powershell...
published: true
date: 2021-09-17T07:49:33.165Z
tags: exchange, powershell, emails
editor: markdown
dateCreated: 2021-09-17T07:49:33.165Z
---

# Introduction

The goal of this script is to mass update emails in Exchange to standardize your user base.


# Script

To use in Powershell IDE:

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