---
title: Searching for a folder with PowerShell
description: Searching for a folder with PowerShell
published: true
date: 2026-10-09T08:00:00.000Z
tags: powershell, folder
editor: markdown
dateCreated: 2026-10-09T08:00:00.000Z
---

# Introduction

The goal of this command is to search for a folder on a drive with PowerShell. It is much faster than Windows Search.

# Usage
Open PowerShell as administrator.
Run the following command and adjust the filter and the path with the name of the folder you are looking for, or part of it with wildcards (*) as shown here:

```powershell
gci -Recurse -Filter "*Photos*" -Directory -ErrorAction SilentlyContinue -Path "D:\"
```
