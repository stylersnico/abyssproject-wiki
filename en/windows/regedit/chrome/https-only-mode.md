---
title: Enabling HTTPS-Only mode in Chrome with Regedit
description: Enabling HTTPS-Only mode in Chrome with Regedit
published: true
date: 2026-10-09T08:00:00.000Z
tags: 
editor: markdown
dateCreated: 2026-10-09T08:00:00.000Z
---

# Introduction
This registry key forces all websites to be upgraded to HTTPS automatically when they support it. Otherwise, a security warning is displayed.

More information: https://chromeenterprise.google/policies/#HttpsOnlyMode

# Script

Put this in a **.reg** file and run it:

```powershell
Windows Registry Editor Version 5.00
[HKEY_LOCAL_MACHINE\Software\Policies\Google\Chrome]
"HttpsOnlyMode"="force_enabled"
```
