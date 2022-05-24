---
title: Backup of a Debian server with Restic
description: Full disk backup of a Debian server with Restic
published: true
date: 2022-05-24T12:19:50.510Z
tags: debian, backup, restic, hetzner
editor: markdown
dateCreated: 2022-05-24T12:18:39.994Z
---

# Introduction

The goal of this is to achieve a partial or complete backup of a Debian server with Restic.

> Here, we will use a [Storage Box](https://www.hetzner.com/storage/storage-box) from Hetzner with SFTP transport and SSH public key authentication.
{.is-info}


# Installation

 
Install Restic :

```bash
apt update
apt install restic
```

# Configuring backup location

> If you use SFTP like me, it's mandatory to do the authentication with a public key, [here is the guide to do it with a Hetzner storage box](https://docs.hetzner.com/robot/storage-box/backup-space-ssh-keys/).
{.is-warning}


Use the following command to set up the repository:
>You should put the same password twice to encrypt the repository with AES256, take note of your password!
{.is-danger}
```bash
restic -r sftp:uXXXXXX@uXXXXXX.your-storagebox.de:/srv/restic-repo init
```

You should have something like this: 
```bash
restic -r sftp:uXXXXXX@uXXXXXX.your-storagebox.de:/srv/restic-repo init
enter password for new repository:
enter password again:
created restic repository 57701a571c at sftp:uXXXXXX@XXXXXX.your-storagebox.de:/srv/restic-repo

Please note that knowledge of your password is required to access
the repository. Losing your password means that your data is
irrecoverably lost.
```


# Backing up the server

To launch a complete backup, you can use the following command:

> In this example, I add an exclusion for the **/var/swap.img** file, you can exclude files and folders with this method.
{.is-info}
```bash
restic -v -r sftp:uXXXXXX@uXXXXXX.your-storagebox.de:/srv/restic-repo backup --exclude /var/swap.img --one-file-system /
```

If you wish to back up a special folder, you can do it like this:

```bash
restic -v -r sftp:uXXXXXX@uXXXXXX.your-storagebox.de:/srv/restic-repo backup /var/www/
```

# Automation

If you wish to automate the process (and you should do it), add this to the end of your command to have more human-readable logs:
```bash
 | cat
```

You also need to pass the repository password to a variable like this:
```bash
export RESTIC_PASSWORD=repo_password
```

For example: 

```bash
export RESTIC_PASSWORD=repo_password
restic -v -r sftp:uXXXXXX@uXXXXXX.your-storagebox.de:/srv/restic-repo backup --exclude /var/swap.img --one-file-system /  | cat
```