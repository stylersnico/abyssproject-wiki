---
title: Configuring a GPU in a virtual machine with DDA
description: Configuring a GPU in a virtual machine with DDA
published: true
date: 2026-10-09T08:00:00.000Z
tags: dda, gpu, nvidia
editor: markdown
dateCreated: 2026-10-09T08:00:00.000Z
---

# Configuring a GPU in a virtual machine with DDA

This procedure passes a GPU directly through to a Windows virtual machine and configures it for use by the system.


# Installing the drivers on the Hyper-V host

First, download and install the latest Datacenter drivers for your card from the following link: https://www.nvidia.com/en-us/drivers/

Restart the server when done.



# Assigning the GPU to the virtual machine

On the physical server, get the "PCI locations" of the card from the server's Device Manager. The information looks like this:

![dda-pcie-location.png](/windows/rds/dda/dda-pcie-location.png)


First, configure Write-Combining on the virtual machine:

```PowerShell
Set-VM CHECRDS01 -GuestControlledCacheTypes $true
```

Then configure the MMIO space of the virtual machine:
```PowerShell
Set-VM -LowMemoryMappedIoSpace 3Gb -VMName CHECRDS01
Set-VM -HighMemoryMappedIoSpace 33280Mb -VMName CHECRDS01
```

Dismount the graphics card from the physical host with the following command:
```PowerShell
Dismount-VMHostAssignableDevice -force -LocationPath "PCIROOT(89)#PCI(0100)#PCI(0000)#PCI(0200)#PCI(0000)#PCI(0000)#PCI(0000)"
```

Now assign the graphics card to the target virtual machine with the following command:
```PowerShell
Add-VMAssignableDevice -VMName CHECRDS01 -LocationPath "PCIROOT(89)#PCI(0100)#PCI(0000)#PCI(0200)#PCI(0000)#PCI(0000)#PCI(0000)"
```

# Configuring the GPU in the virtual machine

Now start your virtual machine and this time, install the GRID drivers, not the standard drivers!
You can find the latest GRID drivers here: https://cloud.google.com/compute/docs/gpus/grid-drivers-table?authuser=0#windows_drivers

Then restart the server and run the following command:
```PowerShell
nvidia-smi.exe
```

Check that your card is detected and that it is in **WDDM** mode, like this:

![dda-wddm.png](/windows/rds/dda/dda-wddm.png)


If it is not, switch it to **WDDM** mode like this:

```
nvidia-smi -i 0 -dm WDDM
```


## Enabling OpenGL on RDS
To make OpenGL available in remote desktop sessions, you need to download and install the following program: https://developer.nvidia.com/nvidia-opengl-rdp

Restart the RDS server.

## Enabling hardware GPU support on RDS

Configure the following policy on the RDS server:
```
"Computer Configuration" > "Administrative Templates" > "Windows Components"
> "Remote Desktop Services" > "Remote Desktop Session Host"
> "Remote Session Environment."
Enable the "Use the hardware default graphics adapter for all Remote Desktop Services sessions" policy.
 ```
 
Restart the RDS server and test with your applications.


# Sources
- https://learn.microsoft.com/en-us/windows-server/virtualization/hyper-v/deploy/deploying-graphics-devices-using-dda
- https://cloud.google.com/compute/docs/gpus/grid-drivers-table?authuser=0#windows_drivers
- https://www.reddit.com/r/nvidia/comments/fx202t/opengl_via_rdp_for_consumer_cards/
