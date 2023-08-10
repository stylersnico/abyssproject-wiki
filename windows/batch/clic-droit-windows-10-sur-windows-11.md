---
title: Remettre le clic droit de Windows 10 sur Windows 11
description: Enlever le nouveau menu au clic-droit
published: true
date: 2023-08-10T12:10:14.726Z
tags: clic-droit, windows 11
editor: markdown
dateCreated: 2023-08-10T12:10:14.726Z
---

# Introduction

Le but de cette ligne de commande est de remettre en place l'ancien menu contextuel après un clic-droit.

# Remise en place

Lancez la ligne suivante dans une ligne de commande (ou dans un script de connexion) :

```powershell
REG.EXE add "HKCU\Software\Classes\CLSID\{86ca1aa0-34aa-4e8b-a509-50c905bae2a2}\InprocServer32" /f /ve
```
