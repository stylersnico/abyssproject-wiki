---
title: Deleting an email from all mailboxes with PowerShell
description: Deleting an email from all mailboxes with PowerShell
published: true
date: 2026-10-09T08:00:00.000Z
tags: exchange, exchange online, compliance
editor: markdown
dateCreated: 2026-10-09T08:00:00.000Z
---

# Introduction
This procedure deletes an email from every mailbox in the tenant, for example after receiving a virus or sending an email by mistake.


# Connecting with the Exchange Online module


Make sure you have version 3 of the Exchange Online module:

```powershell
Get-InstalledModule -Name ExchangeOnlineManagement

Version              Name                                Repository           Description
-------              ----                                ----------           -----------
3.2.0                ExchangeOnlineManagement            PSGallery            This is a General Availability (GA) rele…
```

Otherwise, update and reinstall the module to make sure everything works, then close and reopen PowerShell:

```powershell
Update-Module -Name ExchangeOnlineManagement
Install-Module -Name ExchangeOnlineManagement -Force
```

Connect to Exchange Online with the following command:
```powershell
Connect-ExchangeOnline -UserPrincipalName admin@tenant.ch
```

Assign yourself the following role:
```powershell
Add-RoleGroupMember "Discovery Management" -member admin@tenant.ch
```

Then connect to Compliance in addition to Exchange (right after the first command):
```powershell
Connect-IPPSSession -UserPrincipalName admin@tenant.ch
```

Finally, assign yourself the following role:
```powershell
Add-eDiscoveryCaseAdmin admin@tenant.ch
```

# Creating a search

> Searches require patience. The command line is the slowest to refresh.
> The online portal gets the results a bit faster.
> The mailboxes themselves are the first to be updated after a deletion, so in an emergency, rely on them.
{.is-warning}

If you are looking for a specific email, search directly by subject like this:
```powershell
$Search=New-ComplianceSearch -Name "RemoveMessage_User" -ExchangeLocation all -ContentMatchQuery '(Received:08/31/2023 00:00..08/31/2023 23:59) AND (from:"User.pleux@toto.mail") AND subject:"Re: Validation des notes de frais*"'
```
You can also search for all emails received on a specific day or date range only:
```powershell
$Search=New-ComplianceSearch -Name "RemoveMessage_User" -ExchangeLocation all -ContentMatchQuery '(Received:08/31/2023 00:00..08/31/2023 23:59) AND (from:"User@toto.mail")'
```

Don't forget to start your search:
```powershell
Start-ComplianceSearch “Search User” 
```


# Tracking the results

You can track the results with the following command:
```powershell
Get-ComplianceSearch  "RemoveMessage_User" | FL
```

If your filter is correct, you will see the emails appear:

```powershell
SuccessResults                        : {Location: User.pleux@toto.mail, Item count: 2, Total size: 939535,
                                        Location: 1@toto.mail, Item count: 2, Total size: 545984,
                                        Location: 2@toto.mail, Item count: 1, Total size: 669325,
                                        Location: 3@toto.mail, Item count: 1, Total size: 668918,
                                        Location: 4@toto.mail, Item count: 1, Total size: 668858,
                                        Location: 5@toto.mail, Item count: 1, Total size: 668818,
                                        Location: 6@toto.mail, Item count: 1, Total size: 668809,
```            

> Emails not matched by the filter will not be affected by any action on the search, even if they are displayed.
{.is-info}


# Deleting the emails

If the search is correct, you can start the **permanent** deletion of the emails with the following command:
```powershell
New-ComplianceSearchAction -SearchName "RemoveMessage_User" -Purge -PurgeType HardDelete
```

You can track the operation with the following command:
```powershell
Get-ComplianceSearchAction | fl
```

> The result will show up in the user mailboxes before anywhere else. Remember that this command is destructive: the email will not go to the recycle bin!
{.is-danger}
