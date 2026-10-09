---
title: Restoring the Windows 10 right-click menu on Windows 11
description: Remove the new right-click menu
published: true
date: 2026-10-09T08:00:00.000Z
tags: right-click, windows 11
editor: markdown
dateCreated: 2026-10-09T08:00:00.000Z
---

# Introduction

The goal of this command is to bring back the old context menu on right-click.

# Restoring the menu

Run the following line in a command prompt (or in a logon script):

```powershell
REG.EXE add "HKCU\Software\Classes\CLSID\{86ca1aa0-34aa-4e8b-a509-50c905bae2a2}\InprocServer32" /f /ve
```
