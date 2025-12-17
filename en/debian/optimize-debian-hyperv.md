---
title: Optimize Debian (and Ubuntu) on Hyper-V
description: Optimize Debian 12 / 13 (and Ubuntu 24.04) performance inside an Hyper-V virtual machine
published: true
date: 2025-12-17T07:44:23.184Z
tags: debian, ubuntu, hyper-v
editor: markdown
dateCreated: 2025-12-17T07:44:23.184Z
---

# Introduction

The goal of this guide is to help optimize the performance of a Debian / Ubuntu OS inside a virtual Hyper-V machine.
> This guide is made for Debian 12/13 and Ubuntu 24.04.
{.is-warning}

To make the most of it performance-wise, your virtual machines need to be in generation 2 (but it will also help for gen1 hardware).


# Installing Cloud / Azure kernel and Hyper-V Daemons

On Debian : 
```bash
apt install linux-image-cloud-amd64 hyperv-daemons -y
```

On Ubuntu : 
```bash
apt install linux-azure
```
> This one is a meta-package that include everything that is needed, including hyperv-daemons.
{.is-info}


# Editing Grub 

We will add kernel optimization to help SCSI disk performance and let the I/O control on the Hyper-V host.

Open grub config: 

```bash
nano /etc/default/grub
```

Comment the existing DEFAULT and add this one: 
```bash
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash scsi_mod.use_blk_mq=1 elevator=noop"
```

Update your grub: 
```bash
update-grub
```

> **What is done:**
> **blk-mq** help parralelizing requests on modern hardware, so on SCSI disk.
> **noop** let the Hyper-V host manage the I/O queue. 
{.is-info}

# Tuning sysctl

Launch the following command to create minimal SYSCTL tuning for the virtual machine: 
```bash
tee /etc/sysctl.d/90-hyperv-tuning.conf >/dev/null << 'EOF'
vm.swappiness = 10
vm.dirty_ratio = 10
vm.dirty_background_ratio = 5
EOF
```

> **What is done:**
> **swapinness** : Prevent swapping on disk except if the free ram is really low.
> **dirty_ratio** : Limit writeback change waiting on RAM. Prevents big write spikes.
> **dirty_background_ratio** : Keep the limit of pdflush/kworker writeback to 5% of free memory. Prevent any application blocking from being lower than the dirty_ratio.
{.is-info}
