---
title: Recherche d'un dossier en Powershell
description: Recherche d'un dossier en Powershell
published: true
date: 2025-05-23T12:07:00.410Z
tags: powershell, folder, dossier
editor: markdown
dateCreated: 2025-05-23T12:07:00.410Z
---

# Introduction

Le but de cette commande est de rechercher un dossier en powershell sur un disque, cela va bien plus vite qu'avec la recherche Windows.

# Utilisation
Lancez un powershell en administrateur.
Lancez la commande suivante et adaptez le filtre et le chemin d'accès pour mettre le nom du dossier que vous cherchez ou une partie avec des * comme ici :

```powershell
gci -Recurse -Filter "*Photos*" -Directory -ErrorAction SilentlyContinue -Path "D:\"
```