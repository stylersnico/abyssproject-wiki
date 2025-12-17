---
title: Optimiser Debian (et Ubuntu) sur Hyper-V
description: Optimiser Debian 12 et 13 (et Ubuntu 24.04) à l'intérieur d'une machine virtuelle Hyper-V
published: true
date: 2025-12-17T07:33:47.238Z
tags: debian, ubuntu, hyper-v
editor: markdown
dateCreated: 2025-12-17T07:33:47.238Z
---

# Introduction

Le but de ce guide est d'optimiser les performances d'un système Debian ou Ubuntu à l'intérieur d'une machine virtuelle Hyper-V.
> Ce guide est valide pour Debian 12, 13 et Ubuntu 24.04.
{.is-warning}

Pour en profiter au maximum, il faut que vos machines virtuelles soient en génération 2 (mais cela aidera aussi en génération 1).


# Installation du kernel Cloud / Azure et du démon Hyper-V

Sur Debian : 
```bash
apt install linux-image-cloud-amd64 hyperv-daemons -y
```

Sur Ubuntu : 
```bash
apt install linux-azure
```
> C'est un méta-package qui installera tout ce qu'il faut, y compris hyperv-daemons.
{.is-info}


# Modification du grub 

On va ajouter des optimisations au chargement du noyau pour mieux utiliser les disques SCSI et laisser le contrôle sur les I/O à l'hôte Hyper-V.

Ouvrez la configuration du grub : 

```bash
nano /etc/default/grub
```

Commentez le DEFAULT existant et ajoutez celui-ci à la place : 
```bash
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash scsi_mod.use_blk_mq=1 elevator=noop"
```

Mettez à jour votre grub : 
```bash
update-grub
```

> **Impacts :**
> **blk-mq** améliore la gestion du parralélisme sur les disques SCSI et sur le matériel moderne.
> **noop** laisse la gestion des I/O à l'Hyper-V plutôt qu'au système Debian, pareil cela améliore les performances sur les systèmes modernes.
{.is-info}

# Tuning sysctl

Lancez la commande suivante pour faire des ajustements minimum pour Hyper-V : 
```bash
tee /etc/sysctl.d/90-hyperv-tuning.conf >/dev/null << 'EOF'
vm.swappiness = 10
vm.dirty_ratio = 10
vm.dirty_background_ratio = 5
EOF
```

> **Impacts :**
> **swapinness** : Empêche le swapping sur le disque sauf si la ram libre est très basse.
> **dirty_ratio** : Limite les changements en attente de writeback sur le disque dans la ram. Empêche les gros pics d'écritures sur les disques
> **dirty_background_ratio** : Garde la limite du writeback pdflush/kworker à 5% de la mémoire. Empêche les blocages d'écriture en étant plus bas que le dirty_ratio
{.is-info}
