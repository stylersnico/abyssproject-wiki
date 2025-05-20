---
title: Configuration d'un GPU dans une machine virtuelle via DDA
description: Configuration d'un GPU dans une machine virtuelle via DDA
published: true
date: 2025-05-20T08:03:31.810Z
tags: dda, gpu, nvidia
editor: markdown
dateCreated: 2025-05-20T08:03:31.810Z
---

# Configuration d'un GPU dans une machine virtuelle via DDA

Cette procédure permets le passage d'un GPU directement dans une machine virtuelle Windows ainsi que sa configuration pour l'utilisation dans un système.


# Installation des drivers sur l'hôte Hyper-V

Téléchargez et installez d'abord les derniers drivers Datacenter pour votre carte depuis le lien suivant : https://www.nvidia.com/fr-fr/drivers/

Redémarrez le serveur à la fin.



# Assignation du GPU à la machine virtuelle

Depuis le serveur physique, récupérez les "PCI locations" de la carte depuis le gestionnaire de périphérique du serveur physique, l'information ressemble à cela : 

![dda-pcie-location.png](/generique/windows/rds-dda/dda-pcie-location.png)


Configurez d'abord le Write-Combining sur la machine virtuelle :

```PowerShell
Set-VM CHECRDS01 -GuestControlledCacheTypes $true
```

Configurez ensuite l'espace MMIO de la machine virtuelle : 
```PowerShell
Set-VM -LowMemoryMappedIoSpace 3Gb -VMName CHECRDS01
Set-VM -HighMemoryMappedIoSpace 33280Mb -VMName CHECRDS01
```

Démontez la carte graphique de l'hôte physique avec la commande suivante : 
```PowerShell
Dismount-VMHostAssignableDevice -force -LocationPath "PCIROOT(89)#PCI(0100)#PCI(0000)#PCI(0200)#PCI(0000)#PCI(0000)#PCI(0000)"
```

Assignez maintenant la carte graphique à la machine virtuelle souhaitée avec la commande suivante :
```PowerShell
Add-VMAssignableDevice -VMName CHECRDS01 -LocationPath "PCIROOT(89)#PCI(0100)#PCI(0000)#PCI(0200)#PCI(0000)#PCI(0000)#PCI(0000)"
```

# Configuration du GPU dans la machine virtuelle

Lancez maintenant votre machine virtuelle et cette fois, installez les drivers GRID et non les drivers standards !
Vous pouvez trouver les derniers drivers GRID ici : https://cloud.google.com/compute/docs/gpus/grid-drivers-table?authuser=0#windows_drivers

Redémarrez ensuite le serveur et lancez la commande suivante 
```PowerShell
nvidia-smi.exe
```

Vérifiez que votre carte est reconnue et qu'elle est bien en mode **WDDM** comme ceci : 

![dda-wddm.png](/generique/windows/rds-dda/dda-wddm.png)


Si ce n'est pas le cas, passez la en mode **WDDM** comme ceci :

```
nvidia-smi -i 0 -dm WDDM
```


## Activation d'OpenGL sur les RDS
Pour que OpenGL soit disponible dans les bureaux à distance, vous devez télécharger et installer le programme suivant : https://developer.nvidia.com/nvidia-opengl-rdp

Redémarrez le RDS.

## Activation du support du GPU hardware sur les RDS

Configurez la police suivante sur les RDS : 
```
"Computer Configuration" > "Administrative Templates" > "Windows Components"
> "Remote Desktop Services" > "Remote Desktop Session Host"
> "Remote Session Environment."
Enable the "Use the hardware default graphics adapter for all Remote Desktop Services sessions" policy.
 ```
 
Redémarrez le RDS et testez avec vos applications.


# Sources
- https://learn.microsoft.com/en-us/windows-server/virtualization/hyper-v/deploy/deploying-graphics-devices-using-dda
- https://cloud.google.com/compute/docs/gpus/grid-drivers-table?authuser=0#windows_drivers
- https://www.reddit.com/r/nvidia/comments/fx202t/opengl_via_rdp_for_consumer_cards/