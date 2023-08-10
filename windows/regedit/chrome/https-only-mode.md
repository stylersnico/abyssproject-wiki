---
title: Activation du mode HTTPS ONLY via Regedit dans Chrome
description: Activation du mode HTTPS ONLY via Regedit dans Chrome
published: true
date: 2023-08-10T11:58:07.361Z
tags: 
editor: markdown
dateCreated: 2023-08-10T11:58:07.361Z
---

# Introduction
Cette clé de registre force l'upgrade automatique de tous les sites vers le protocole HTTPS si ceux-ci supportent cela, sinon un avertissement de sécurité sera affiché.

Pour plus d'informations : https://chromeenterprise.google/policies/#HttpsOnlyMode

# Script

Mettez cela dans un fichier **.reg** et exécutez-le : 

```powershell
Windows Registry Editor Version 5.00
[HKEY_LOCAL_MACHINE\Software\Policies\Google\Chrome]
"HttpsOnlyMode"="force_enabled"
```
