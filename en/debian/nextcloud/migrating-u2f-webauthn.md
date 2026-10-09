---
title: Migrating from U2F to WebAuthn on Nextcloud via CLI
description: Migrating from U2F to WebAuthn on Nextcloud via CLI
published: true
date: 2026-10-09T08:00:00.000Z
tags: nextcloud, u2f, webauthn
editor: markdown
dateCreated: 2026-10-09T08:00:00.000Z
---

# Introduction

The goal of this procedure is to move from the Nextcloud U2F extension to the WebAuthn app for hardware security key support.

We will also migrate all devices registered by users so that the change is transparent for them.

- Old app: https://apps.nextcloud.com/apps/twofactor_u2f
- New app: https://apps.nextcloud.com/apps/twofactor_webauthn


# Command-line migration

Go to the folder that contains your Nextcloud installation:
```bash
cd /var/www/nextcloud/files.nicolas-simond.ch
```

Install the WebAuthn app and migrate all user devices to the new standard:
```bash
sudo -u nextcloud php occ app:install twofactor_webauthn
sudo -u nextcloud php occ twofactor_webauthn:migrate-u2f -nvv --all
```

You will get the following output:
```bash
Migrating all devices of all users ...
Migrating devices of user XXX
Migrating devices of user XXX
Migrating devices of user XXX
Migrating devices of user XXX
```

Log in and check that everything works.
If everything is OK, you can remove the old app:

```bash
sudo -u nextcloud php occ app:disable twofactor_u2f
sudo -u nextcloud php occ twofactorauth:cleanup u2f
sudo -u nextcloud php occ app:remove twofactor_u2f
```

## Source

- Trumme@reddit: https://www.reddit.com/r/NextCloud/comments/wyuwze/comment/ilyx67t/?utm_source=share&utm_medium=web2x&context=3
