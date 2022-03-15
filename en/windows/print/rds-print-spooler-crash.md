---
title: Repairing frozen print server on RDS 2016 / 2019
description: Repairing the print spooler that froze without any reason or error
published: true
date: 2022-03-15T10:20:58.969Z
tags: windows, printing
editor: markdown
dateCreated: 2022-03-15T10:20:58.969Z
---

# Introduction

The print spooler was frozing completely without any reason or error message and without any printer installed on Windows Server 2016 / 2019 with remote desktop role.

 
# Resolving the problem

Open powershell as admin and launch the following commands: 

```powershell
net stop spooler
Remove-Item -Path "HKLM:\Software\Microsoft\Windows NT\CurrentVersion\Print\Providers\Client Side Rendering Print Provider*" –Recurse
net start spooler
```
 

 
## Sources

- https://docs.microsoft.com/en-us/answers/questions/451102/server-2016-rds-print-spooler-issues.html