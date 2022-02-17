---
title: Azure Ad Sync won't start: Repair error code 575
description: Fix the Azure AD Sync service that won't start after upgrade
published: true
date: 2022-02-17T07:47:05.772Z
tags: azure, microsoft, adsync, office 365
editor: markdown
dateCreated: 2022-02-17T07:30:22.574Z
---

# Before starting

The goal here is to fix the Microsoft Azure AD Sync service that stays stuck in "Starting" after rebooting the server where the service is installed (mostly after auto-update I think).

![azureadsync-error-575-1.png](/azure/azureadsync-error-575-1.png.webp)


# Checking the error

Check that you have the following message in your event viewer: 

```
Windows API call WaitForMultipleObjects returned error code: 575. Windows system error message is: {Application Error}
The application was unable to start correctly (0x%lx). Click OK to close the application.
```

![azureadsync-error-575-2.png](/azure/azureadsync-error-575-2.png.webp)

# Fixing the error

If your service is blocked is "starting" kill the service first by killing the process "AD-IAM-HybridSync master":
![azureadsync-error-575-3.png](/azure/azureadsync-error-575-3.png.webp)

Now you need to copy to following files:
```
C:\Program Files\Microsoft SQL Server\150\LocalDB\Binn\Templates\model.mdf
C:\Program Files\Microsoft SQL Server\150\LocalDB\Binn\Templates\modellog.ldf
```

Now, copy these two files to the following folder if you use a dedicated service account:
```
C:\Users\ADSyncxxxxx$\AppData\Local\Microsoft\Microsoft SQL Server Local DB\Instances\ADSync2019.
```

Or this folder:
```
C:\Windows\ServiceProfiles\ADSync\AppData\Local\Microsoft\Microsoft SQL Server Local DB\Instances\ADSync2019
```

When the files are overwritten, just restart the service:
![azureadsync-error-575-3.png](/azure/azureadsync-error-575-4.png.webp)