---
title: Export des utilisateurs et des emails en Powershell
description: Export des utilisateurs et des emails en Powershell
published: true
date: 2023-02-10T08:16:42.584Z
tags: powershell, email, ad, active directory
editor: markdown
dateCreated: 2023-02-10T08:16:42.584Z
---

# Script

Dans une commande Powershell, lancez la commande suivante en modifiant votre Search Base pour extraire une OU précise sous la forme "Nom d'affichage, Adresse Email" :

```powershell
Get-ADUser -Filter * -SearchBase "OU=Administration,OU=Antaes USER and Group,DC=antaes,DC=ch" -Properties DisplayName, EmailAddress | select DisplayName, EmailAddress | Export-CSV C:\Email_Addresses.csv
```

# Export automatique et envoi par email

Il est possible de faire un export automatique par email avec ce genre de script (fait pour Antaes à la base) : 

```powershell
#####Define Variables#####

$fromaddress = “it@antaes.ch”
$toaddress = “backoffice@antaes.ch”
$Subject = “Export AD Antaes”
$body = "Export automatique de l'AD Antaes"
$attachment = “C:\_Scripts\Export.Zip”
$smtpserver = “outbound-eu1.ppe-hosted.com”

##########################

# Clean everything

Remove-Item -Path "C:\_Scripts\Export.Zip"
Remove-Item -Path "C:\_Scripts\Consultants_Email_Addresses.csv"
Remove-Item -Path "C:\_Scripts\Administration_Email_Addresses.csv"

# Exports all users

Get-ADUser -Filter * -SearchBase "OU=Administration,OU=Antaes USER and Group,DC=antaes,DC=ch" -Properties UserPrincipalName, GivenName, Surname, EmailAddress | select UserPrincipalName, GivenName, Surname, EmailAddress | Export-CSV "C:\_Scripts\Administration_Email_Addresses.csv"
Get-ADUser -Filter * -SearchBase "OU=Consultants,OU=Antaes USER and Group,DC=antaes,DC=ch" -Properties UserPrincipalName, GivenName, Surname, EmailAddress | select UserPrincipalName, GivenName, Surname, EmailAddress | Export-CSV "C:\_Scripts\Consultants_Email_Addresses.csv"

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