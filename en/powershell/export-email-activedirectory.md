---
title: Export users and emails in PowerShell
description: Export users and emails in PowerShell
published: true
date: 2023-02-10T08:19:52.502Z
tags: powershell, email, ad, active directory
editor: markdown
dateCreated: 2023-02-10T08:19:52.502Z
---

# Script

In a powershell prompt, edit the following command with your search base with the form "Display Name, EmailAddress":

```powershell
Get-ADUser -Filter * -SearchBase "OU=Administration,OU=USER and Group,DC=domain,DC=ch" -Properties DisplayName, EmailAddress | select DisplayName, EmailAddress | Export-CSV C:\Email_Addresses.csv
```

# Automated export and email report

You can automate the export with this kind of script:

```powershell
#####Define Variables#####

$fromaddress = “it@domain.ch”
$toaddress = “backoffice@domain.ch”
$Subject = “Export AD domain”
$body = "Export automatique de l'AD domain"
$attachment = “C:\_Scripts\Export.Zip”
$smtpserver = “outbound-eu1.ppe-hosted.com”

##########################

# Clean everything

Remove-Item -Path "C:\_Scripts\Export.Zip"
Remove-Item -Path "C:\_Scripts\Consultants_Email_Addresses.csv"
Remove-Item -Path "C:\_Scripts\Administration_Email_Addresses.csv"

# Exports all users

Get-ADUser -Filter * -SearchBase "OU=Administration,OU=domain USER and Group,DC=domain,DC=ch" -Properties UserPrincipalName, GivenName, Surname, EmailAddress | select UserPrincipalName, GivenName, Surname, EmailAddress | Export-CSV "C:\_Scripts\Administration_Email_Addresses.csv"
Get-ADUser -Filter * -SearchBase "OU=Consultants,OU=domain USER and Group,DC=domain,DC=ch" -Properties UserPrincipalName, GivenName, Surname, EmailAddress | select UserPrincipalName, GivenName, Surname, EmailAddress | Export-CSV "C:\_Scripts\Consultants_Email_Addresses.csv"

# Compress files 

$compress = @{
LiteralPath= "C:\_Scripts\Administration_Email_Addresses.csv", "C:\_Scripts\Consultants_Email_Addresses.csv"
CompressionLevel = "Fastest"
DestinationPath = "C:\_Scripts\Export.Zip"
}
Compress-Archive -Force @compress

# Send the compressed file

$message = new-object System.Net.Mail.MailMessage
$message.From = $fromaddress
$message.To.Add($toaddress)
$message.IsBodyHtml = $True
$message.Subject = $Subject
$attach = new-object Net.Mail.Attachment($attachment)
$message.Attachments.Add($attach)
$message.body = $body
$smtp = new-object Net.Mail.SmtpClient($smtpserver)
$smtp.Send($message)
```