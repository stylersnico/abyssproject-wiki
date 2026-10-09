---
title: Converting VHD and VHDX disks to fixed-size VHDX
description: Converting VHD and VHDX disks to fixed-size VHDX in Hyper-V
published: true
date: 2026-10-09T08:00:00.000Z
tags: powershell, hyper-v, vhd, vhdx
editor: markdown
dateCreated: 2026-10-09T08:00:00.000Z
---

# Introduction

The goal of this script is to automatically convert the virtual disks of every virtual machine on a Hyper-V server.

> Shut down and back up your virtual machines before doing anything
{.is-danger}


# Script

Import the Hyper-V module in PowerShell:

```powershell
import-module hyper-v -requiredversion 1.1
```

This script automatically gets the list of all virtual machines and all disks attached to them:

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

If you only want to do this on one or more VMs based on their names, you can adapt the script like this:

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

> If you had .VHD disks, they have been automatically converted to the VHDX format, so you will need to point your virtual machine configurations to the new disk path.
{.is-warning}
