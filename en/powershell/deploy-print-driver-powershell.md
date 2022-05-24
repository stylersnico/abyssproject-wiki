---
title: Deploying print driver via Powershell
description: Deploying print driver via Powershell after Printnightmare
published: true
date: 2022-05-24T11:47:25.592Z
tags: powershell, printer
editor: markdown
dateCreated: 2022-05-24T11:47:25.592Z
---

# Introduction

The goal of this script is to deploy a print driver on AD joined computers after the Printnightmare era.


# Script

Download and extract your print driver somewhere on your server like this:

> D:\Data\Deploy\v4_02_PCL6_1712a\PCL6\64bit

Next, install it on your print driver to get the driver exact name and the final location of the driver information file:
> "SHARP Driver(v4) PCL6"
> .
>C:\Windows\System32\DriverStore\FileRepository\su06menu.inf_amd64_aff6b7d3b8d3aed7\su06menu.inf

Finally, prepare the script who will deploy and install the driver on all computers:

> "C:\computers.txt" must contain the list of all computers that will receive the driver.
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