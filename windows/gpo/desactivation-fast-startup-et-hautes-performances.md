---
title: Désactivation du Fast startup via GPO
description: Et application du mode hautes performances pour enlever le mode hybride HP
published: true
date: 2023-08-10T12:03:55.662Z
tags: gpo, fast boot, démarrage rapide, hautes performances
editor: markdown
dateCreated: 2023-08-10T12:03:55.662Z
---

# Introduction

Le but de ces paramètres de GPO est de désactiver le démarrage rapide du PC.
On configure aussi le mode hautes performances via GPO pour désactiver le plan d'alimentation hybride de HP qui cause des problèmes de réseau et des ralentissements.

> On parle de ce paramètre :
> ![fast-startup-01.png](/windows/gpo/fast-startup/fast-startup-01.png)
{.is-info}
# Stratégie de groupe

Voici les paramètres que vous devez appliquer pour aboutir au résultat souhaité :

![fast-startup-02.png](/windows/gpo/fast-startup/fast-startup-02.png)