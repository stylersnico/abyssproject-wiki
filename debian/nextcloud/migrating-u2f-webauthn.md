---
title: Migration de U2F à Webauthn sur Nextcloud en CLI
description: Migration de U2F à Webauthn sur Nextcloud en CLI
published: true
date: 2023-09-01T08:10:41.593Z
tags: nextcloud, u2f, webauthn
editor: markdown
dateCreated: 2023-09-01T08:10:41.593Z
---

# Introduction

Le but de cette procédure est de passer de l'extension Nextcloud U2F à l'application WebAuthn pour le support des clés physiques d'authentification.

Nous migrerons également tous les périphériques enregistrés chez les utilisateurs afin que cela soit transparent pour eux.

- Ancienne application : https://apps.nextcloud.com/apps/twofactor_u2f
- Nouvelle application : https://apps.nextcloud.com/apps/twofactor_webauthn


# Migration en ligne de commande

Mettez-vous dans le dossier qui contient votre installation Nextcloud : 
```bash
cd /var/www/nextcloud/files.nicolas-simond.ch
```

Installez l'application pour le webauthn et migrez tous les périphériques utilisateurs sur le nouveau standard : 
```bash
sudo -u nextcloud php occ app:install twofactor_webauthn
sudo -u nextcloud php occ twofactor_webauthn:migrate-u2f -nvv --all
```

Vous aurez la sortie suivante : 
```bash
Migrating all devices of all users ...
Migrating devices of user XXX
Migrating devices of user XXX
Migrating devices of user XXX
Migrating devices of user XXX
```

Connectez-vous et vérifiez que tout marche.
Si tout est OK, vous pourrez supprimer l'ancienne application : 

```bash
sudo -u nextcloud php occ app:disable twofactor_u2f
sudo -u nextcloud php occ twofactorauth:cleanup u2f
sudo -u nextcloud php occ app:remove twofactor_u2f
```

## Source

- Trumme@reddit : https://www.reddit.com/r/NextCloud/comments/wyuwze/comment/ilyx67t/?utm_source=share&utm_medium=web2x&context=3