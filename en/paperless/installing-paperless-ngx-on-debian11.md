---
title: Installing Paperless-NGX on Debian 11
description: Installing Paperless-NGX on Debian 11 with dedicated user
published: true
date: 2022-05-27T06:22:13.197Z
tags: paperless, dms, edm
editor: markdown
dateCreated: 2022-05-27T06:19:48.473Z
---

# Introduction

The goal here is to set up an Electronic Document Management system with the Open source product Paperless-NGX.

> We won't talk about external access here since the system doesn't have two factor authentication right now. 
{.is-info}


# Docker Installation

 
Install the prerequisites:

```bash
apt-get update && apt-get install ca-certificates curl gnupg lsb-release sudo -y
```

Set-up Docker Repository and Docker: 
```bash
curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian $(lsb_release -cs) stable" |  tee /etc/apt/sources.list.d/docker.list > /dev/null

apt-get update && apt-get install docker-ce docker-ce-cli containerd.io -y
```

Set-up Docker Compose V1:
```bash
curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
 chmod +x /usr/local/bin/docker-compose
ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose

docker-compose --version
```

# Creating Paperless User

Create the Linux user for Paperless:
```bash
adduser paperless
```

Add it to the mandatory group and connect to it: 
```bash
usermod -aG sudo paperless
usermod -aG docker paperless
su - paperless
```


# Paperless Installation

Launch the following script to install Paperless:
```bash
bash -c "$(curl -L https://raw.githubusercontent.com/paperless-ngx/paperless-ngx/master/install-paperless-ngx.sh)"
```


Some options will be needed. Personally, I like to keep media, data and database off a docker volume, directly on the file system for more simple backups:

> Enable Apache Tika if you would like to add more files than simple PDF into your EDM (Office documents for example).
{.is-info}


```bash
Summary
=======

Target folder: /home/paperless/paperless-ngx
Consume folder: /home/paperless/paperless-ngx/consume
Media folder: /home/paperless/documents
Data folder: /home/paperless/data
Database (postgres) folder: /home/paperless/database

Port: 8000
Database: postgres
Tika enabled: yes
OCR language: fra
User id: 1001
Group id: 1001
Timezone : Europe/Paris

Paperless username: paperless
Paperless email: paperless@domain.com
```

> At the end of the set-up, Paperless will be available from this URL: http://IP_ADDRESS:8000/
{.is-success}


## Automatic update of the system

If you would like to automate the update process of the system (you should do it), install Watchtower:
- https://wiki.abyssproject.net/en/docker/watchtower-docker-update