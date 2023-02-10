---
title: Clean the tape before Veeam Backup
description: Clean the tape before Veeam Backup
published: true
date: 2023-02-10T08:26:18.250Z
tags: backup, powershell, veeam, tape
editor: markdown
dateCreated: 2023-02-10T08:26:18.250Z
---

# Introduction

The goal of this script is to clean the tape before a veeam backup to free up the old data.

# Getting the tape library name

Open Veeam and take note of the tape reader:

![veeam-tape-erase-before-backup-01.png](/scripts/powershell/veeam-tape-erase-before-backup-01.png)

# Script

This script inventory the tape inside the reader and do a quick erase of the data on the tape:

```powershell
Start-VBRTapeInventory -library "HP Ultrium 7-SCSI"
$tape = Get-VBRTapeMedium –library "HP Ultrium 7-SCSI"
Erase-VBRTapeMedium –Medium $tape
sleep 120
```

# Using it

Save this script inside a powershell file.
Open the job and go into the advanced option:

![veeam-tape-erase-before-backup-02.png](/scripts/powershell/veeam-tape-erase-before-backup-02.png)

Configure your script like this: 

![veeam-tape-erase-before-backup-03.png](/scripts/powershell/veeam-tape-erase-before-backup-03.png)