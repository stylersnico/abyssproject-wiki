---
title: Conversion des disques VHD et VHDX en VHDX de taille fixe
description: Conversion des disques VHD et VHDX en VHDX de taille fixe dans Hyper-V
published: true
date: 2023-08-10T11:59:35.891Z
tags: powershell, hyper-v, vhd, vhdx
editor: markdown
dateCreated: 2023-08-10T11:59:35.891Z
---

# Introduction

Le but de ce script est de convertir automatiquement les disques virtuels de toutes les machines présentes sur un serveur Hyper-V.

> Eteignez et sauvegardez vos machines virtuelles avant toute opération
{.is-danger}


# Script

Importez le module Hyper-V dans Powershell : 

```powershell
import-module hyper-v -requiredversion 1.1
```

Ce script récupère automatiquement la liste de toutes les machines et de tous les disques qui y sont rattachés : 

```powershell
$GetVM = get-vm -computername localhost  | select name -ExpandProperty name
Foreach ($vmname in $GetVM) {
	Foreach ($disk in Get-VMHardDiskDrive $vmname | select Path -ExpandProperty Path) {
		echo $disk
		$NewDisk = "$disk.new.vhdx"
		Convert-VHD -Path $disk -DestinationPath $NewDisk -VHDType Fixed
		Get-Acl "$disk" | Set-Acl "$NewDisk"
		Remove-Item -Path "$disk"
		Move-Item -Path "$NewDisk" -Destination "$disk"
	}
}
```

Si vous souhaitez uniquement faire cela sur une ou plusieurs VMs selon leurs noms, vous pouvez adapter le script comme ceci : 

```powershell
$GetVM = get-vm -computername localhost  | select name -ExpandProperty name | Where-Object {$_.name  -Like "*automate*"}

Foreach ($vmname in $GetVM) {
	Foreach ($disk in Get-VMHardDiskDrive $vmname | select Path -ExpandProperty Path) {
		echo $disk
		$NewDisk = "$disk.new.vhdx"
		Convert-VHD -Path $disk -DestinationPath $NewDisk -VHDType Fixed
		Get-Acl "$disk" | Set-Acl "$NewDisk"
		Remove-Item -Path "$disk"
		Move-Item -Path "$NewDisk" -Destination "$disk"
	}
}
```

> Si vous aviez des disques en .VHD, ils ont automatiquement étés convertis vers le format VHDX, vous devrez donc indiquer le chemin vers le nouveau disque dans les configurations de vos machines virtuelles.
{.is-warning}
