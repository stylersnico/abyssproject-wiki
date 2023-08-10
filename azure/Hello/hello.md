---
title: Azure suppresion de la connexion avec Hello
description: Supprimer l'authentification Hello sur le poste et avec l'utilisateur
published: true
date: 2023-08-10T12:13:35.899Z
tags: azure, hello
editor: markdown
dateCreated: 2023-08-10T12:13:35.899Z
---

# Introduction

Le but ici est de forcer la suppression de l'authentification avec Hello sur le poste et sur l'utilisateur, lors de l'activation d'un Azure active directory

# Lancer Microsoft Management Console

Lancer la commande mmc : 

![commande-mmc.png](/azure/windows-hello/commande-mmc.png)

Cela vous ouvre par la suite une Console ou il va falloir ajouter les GPO local :

![console-fichier.png](/azure/windows-hello/console-fichier.png)
![gpo.png](/azure/windows-hello/gpo.png)

Puis la selectionner et cocher la case "finish"

# Désactiver La GPO Hello

Pour désactiver La GPO Hello il faut aller dans les paramètres du PC en local et celui de l'utilisateur:

Faire pour les deux les modifications
 
![localcomputerpolicy.png](/azure/windows-hello/localcomputerpolicy.png)![localcomputerpolicy_-_copy.png](/azure/windows-hello/localcomputerpolicy_-_copy.png)

Suivre le chemin pour acceder :
```
Administrative Templates\Windows Components\Windows Hello for Business
```

Sélectionner par la suite Use Windows Hello for Business

![windows-hello-business.png](/azure/windows-hello/windows-hello-business.png)

Et cocher la case Disabled pour appliquer les modifications 

![console-hello.png](/azure/windows-hello/console-hello.png)

Redémarrer le poste





