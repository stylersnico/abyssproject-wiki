---
title: Script pour appliquer une limite de stockage aux SharePoint
description: Script pour appliquer une limite de stockage aux SharePoint
published: true
date: 2025-05-20T07:48:57.898Z
tags: sharepoint, quota
editor: markdown
dateCreated: 2025-05-20T07:48:57.898Z
---

# Appliquer une limite de stockage aux Sharepoint

Dans cet exemple, nous verrons comment mettre un quota sur tous les sharepoints inférieurs à 40Gb et d'appliquer un quota de 40Go avec une alerte fixé à 30Go (75% du quota).

## Activer la limite de stockage

Pour se faire, il faut déjà activer les quotas pour tous les sharepoints 
Dans la console admin Sharepoint, aller dans **Settings** et activer la limite sharepoint, passer le en **manuel**.

Lorsque vous avez fait ceci, le quota s'applique à tous les sharepoints.
Le quota appliqué est de **25To**.

## Modifier toutes les limites en un script

Comme certains Sharepoint dépasse déjà cette limite, nous allons exécuter ce script qui permet d'appliquer le quota à tous les sharepoints inférieurs à 40Go.

````Bash
Connect-SPOService -Url ....

# Obtenir la liste de tous les sites
$sites = Get-SPOSite -Limit All

# Filtrer les sites avec une taille inférieure à 40 Go
$filteredSites = $sites | Where-Object { $_.StorageUsageCurrent -lt 40000 }

# Boucle à travers chaque site filtré pour configurer la limite et l'alerte
foreach ($site in $filteredSites) {
    # Configurer la limite de stockage à 40 Go et l'alerte à 30 Go
    Set-SPOSite -Identity $site.Url -StorageQuota 40000 -StorageQuotaWarningLevel 30000
    
    Write-Host "La limite de stockage a été configurée pour le site : $($site.Url)"
}

Write-Host "Configuration des quotas terminée pour tous les sites inférieurs à 40 Go."
````

Vous aurez un apperçu de tous les sharepoints ou le quota s'est appliqué.