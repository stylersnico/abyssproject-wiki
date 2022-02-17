---
title: Azure Ad Sync ne démarre pas : Réparer l'erreur 575
description: Réparer le service Azure ADSync qui ne redémarre pas après une mise à jour 
published: true
date: 2022-02-17T07:55:45.872Z
tags: azure, microsoft, adsync, office 365
editor: markdown
dateCreated: 2022-02-17T07:55:33.550Z
---

# Introduction

Le but ici est de réparer le service Microsoft Azure AD Sync qui reste bloqué sur "Starting" après un redémarrage du serveur (probablement après une mise à jour automatique de l'agent).

![azureadsync-error-575-1.png](/azure/azureadsync-error-575-1.png.webp)


# Vérification de l'erreur

Vérifiez que vous avez l'erreur suivante dans le gestionnaire d'évenement : 

```
Windows API call WaitForMultipleObjects returned error code: 575. Windows system error message is: {Application Error}
The application was unable to start correctly (0x%lx). Click OK to close the application.
```

![azureadsync-error-575-2.png](/azure/azureadsync-error-575-2.png.webp)

# Résolution de l'erreur

Si votre service est encore bloqué en "Starting", tuez-le dans le gestionnaire des tâches : 
![azureadsync-error-575-3.png](/azure/azureadsync-error-575-3.png.webp)

Maintenant, copiez les fichiers suivants :
```
C:\Program Files\Microsoft SQL Server\150\LocalDB\Binn\Templates\model.mdf
C:\Program Files\Microsoft SQL Server\150\LocalDB\Binn\Templates\modellog.ldf
```

Et placez ces fichiers dans le dossier suivant si vous utilisez un compte de service dédié pour la synchronisation :
```
C:\Users\ADSyncxxxxx$\AppData\Local\Microsoft\Microsoft SQL Server Local DB\Instances\ADSync2019\
```

Ou ce dossier :
```
C:\Windows\ServiceProfiles\ADSync\AppData\Local\Microsoft\Microsoft SQL Server Local DB\Instances\ADSync2019\
```

Lorsque les fichiers sont remplacés, redémarrez le service:
![azureadsync-error-575-3.png](/azure/azureadsync-error-575-4.png.webp)

## Source

Merci à PatD442: https://www.reddit.com/r/sysadmin/comments/rxkd7m/has_your_azure_ad_connect_been_unable_to_start/

