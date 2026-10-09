---
title: Optimizing RDS performance with Regedit
description: Optimizing RDS performance with Regedit
published: true
date: 2026-10-09T08:00:00.000Z
tags: rds, regedit
editor: markdown
dateCreated: 2026-10-09T08:00:00.000Z
---

# Introduction
This registry file improves the performance of an RDS server.


# Script

Put this in a **.reg** file and run it:

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


# Going further

- https://github.com/maxprehl/TurboRemoteFX


# Sources

- https://www.reddit.com/r/sysadmin/comments/fv7d12/pushing_remote_fx_to_its_limits/?context=3
