---
title: Deploiement de drivers d'imprimante par Powershell
description: Déploiement de drivers par Powershell après Printnightmare
published: true
date: 2022-05-24T11:42:24.323Z
tags: powershell, imprimante
editor: markdown
dateCreated: 2022-05-24T11:42:24.323Z
---

# Introduction

Le but de ce script est de déployer et d'enregistrer des drivers d'impressions sur les PC d'un parc AD après la faille Printnightmare.


# Script

Récupérez et extrayez d'abord votre pilote d'impression quelque part sur votre serveur, par exemple :

> D:\Data\Deploy\v4_02_PCL6_1712a\PCL6\64bit

Ensuite, installez-le sur le serveur d'impression afin d'isoler le nom du pilote et le fichier d'information du pilote :
> "SHARP Driver(v4) PCL6"
> .
>C:\Windows\System32\DriverStore\FileRepository\su06menu.inf_amd64_aff6b7d3b8d3aed7\su06menu.inf

Ensuite, préparez un script de déploiement qui copiera et installera le pilote sur tous les postes :

> "C:\computers.txt" doit contenir une liste des ordinateurs qui recevront le pilote.
{.is-info}


```powershell
$computers = Get-Content "C:\computers.txt"

foreach ($computer in $computers) {
Copy-Item -Path "D:\Data\Deploy\v4_02_PCL6_1712a\PCL6\64bit*" -Destination "\\$computer\c$\temp\64bit\" -Recurse -Force
    $session = New-PSSession -ComputerName $computer
    Invoke-Command -Session $session -ScriptBlock {
        pnputil.exe /a "C:\temp\64bit\su06menu.inf"
        Add-PrinterDriver -Name "SHARP Driver(v4) PCL6" -InfPath "C:\Windows\System32\DriverStore\FileRepository\su06menu.inf_amd64_aff6b7d3b8d3aed7\su06menu.inf"
    }
Remove-PSSession $session
}
```