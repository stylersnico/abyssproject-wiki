---
title: Nettoyer l'historique d'un repository GIT
description: Supprimer tout l'historique d'un repository
published: true
date: 2022-02-17T08:47:03.307Z
tags: git
editor: markdown
dateCreated: 2022-02-17T08:47:03.307Z
---

# Introduction

Le but ici est de supprimer tout l'historique sur un repository.

C'est pratique dans le cas où vous auriez par exemple mis en ligne des informations critiques et que vous souhaitez en effacer les traces.

# Suppression de l'historique d'un repository

> Soyez sûr d'avoir une bonne connexion internet, vous allez télécharger tout le repository, le supprimer et remettre les fichiers en ligne.
{.is-warning}


Connectez-vous d'abord en ligne de commande à votre repository.

> Si l'historique est sur une branche autre que "main", remplacez le nom dans les exemples sinon vous ne supprimez rien.
{.is-danger}

Lancez un checkout de l'existant : 
```
git checkout --orphan latest_branch
```

Ajoutez tous les fichiers :
```
git add -A
```

Faites un commit des changements avec la raison de la suppression : 
```
git commit -am "Deleting history"
```

Supprimez la branche existante :

```
git branch -D main
```



Renommez la branche temporaire avec l'ancien nom : 
```
git branch -m main
```

Remettez maintenant le repository à jour :
```
git push -f origin main
```


## Source
Merci à : https://stackoverflow.com/a/26000395