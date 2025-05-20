---
title: Optimisation de performances sur les RDS via Regedit
description: Optimisation de performances sur les RDS via Regedit
published: true
date: 2025-05-20T07:43:32.980Z
tags: rds, regedit
editor: markdown
dateCreated: 2025-05-20T07:43:32.980Z
---

# Introduction
Ce fichier de registre permets d'améliorer les performances d'un RDS.


# Script

Mettez cela dans un fichier **.reg** et exécutez-le : 

```powershell
Windows Registry Editor Version 5.00

;Sets 60 FPS limit on RDP.
;Source: https://support.microsoft.com/en-us/help/2885213/frame-rate-is-limited-to-30-fps-in-windows-8-and-windows-server-2012-r

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations]

"DWMFRAMEINTERVAL"=dword:0000000f

;Increase Windows Responsivness
;Source:https://www.reddit.com/r/killerinstinct/comments/4fcdhy/an_excellent_guide_to_optimizing_your_windows_10/

[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Multimedia\SystemProfile]

"SystemResponsiveness"=dword:00000000

;Sets the flow control for Display vs Channel Bandwidth (aka RemoteFX devices, including controllers.)

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\TermDD]

"FlowControlDisable"=dword:00000001

"FlowControlDisplayBandwidth"=dword:0000010

"FlowControlChannelBandwidth"=dword:0000090

"FlowControlChargePostCompression"=dword:00000000

;Removes the artificial latency delay for RDP.

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp]

"InteractiveDelay"=dword:00000000

;Disables Windows Network Throtelling.

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\LanmanWorkstation\Parameters]

"DisableBandwidthThrottling"=dword:00000001

;Enables large MTU packets.

"DisableLargeMtu"=dword:00000000

;Enable the WDDM Drivers.

[HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows NT\Terminal Services]

"fEnableWddmDriver"=dword:00000001
```


# Pour aller plus loin 

- https://github.com/maxprehl/TurboRemoteFX


# Sources

- https://www.reddit.com/r/sysadmin/comments/fv7d12/pushing_remote_fx_to_its_limits/?context=3