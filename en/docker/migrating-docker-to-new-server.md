---
title: Migrating a Docker install to a new server
description: Migration of all containers and images with their data on a new server
published: true
date: 2022-05-19T16:47:37.451Z
tags: docker
editor: markdown
dateCreated: 2022-05-19T16:47:37.451Z
---

# Introduction

The goal of this document is to move a full Docker installation with all the data on it to a new server without any loss.

> Ideally, you should have the same Docker release on the two servers so the migration can complete without problem.
{.is-warning}

> Right now, don't install Docker on the new server! 
{.is-danger}


# List the existing mounting points

First, we need to check if we have any mounting point on the local file system with this command:

```bash
docker ps -q | xargs docker inspect -f '{{.Name}} : {{ range .HostConfig.Binds }} {{.}}{{end}}'
```

Here is what I have on my installation: 
```bash
/commento_db_1 :  commento_postgres_data_volume:/var/lib/postgresql/data:rw
/db :  /etc/wiki/.db-secret:/etc/wiki/.db-secret:ro pgdata:/var/lib/postgresql/data
/wiki :  /etc/wiki/.db-secret:/etc/wiki/.db-secret:ro
/watchtower :  /var/run/docker.sock:/var/run/docker.sock
/commento_server_1 :
/wiki-update-companion :  /var/run/docker.sock:/var/run/docker.sock:ro
```

We can see that the only data I have outside a volume is for the wiki:

```bash
/db :  /etc/wiki/.db-secret:/etc/wiki/.db-secret:ro pgdata:/var/lib/postgresql/data
/wiki :  /etc/wiki/.db-secret:/etc/wiki/.db-secret:ro
```

> /etc/wiki/.db-secret



You can ignore the Docker listening socket, we don't need it:

> /var/run/docker.sock




# Moving the data with Rsync

> By default in Debian 11, all Docker datas sits in **/var/lib/docker**, this may be different in your distribution.
{.is-info}

> It's important that you keep the same right on old and new server, if you used to have a special user for docker, create it before the move.
{.is-warning}

Stop docker on the old server: 
```bash
docker stop $(docker ps -a -q)
systemctl stop docker.socket && systemctl stop docker.service
systemctl disable docker.socket && systemctl disable docker.service
```


The simplest way is to grab the data directly from the new server with rsync:

```bash
rsync -avp --progress root@old_server_ip:/var/lib/docker/ /var/lib/docker/
rsync -avp --progress root@old_server_ip:/etc/wiki/.db-secret /etc/wiki/.db-secret
```

# Re-installing Docker
> 
> This part is only for Debian 11 as an example
{.is-info}

Configure Docker repository: 
```bash
apt-get update && apt-get install ca-certificates curl gnupg lsb-release -y
```

Install Docker GPG key: 
```bash
curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

Install Docker:
```bash
apt-get update && apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin -y
```

# Starting Docker and testing

Now restart the containers:

```bash
systemctl start docker.socket && systemctl start docker.service
docker start $(docker ps -a -q)
```


Check that they are running: 

```bash
docker ps -a
```

Here is what I have on my side:
```bash
CONTAINER ID   IMAGE                                   COMMAND                  CREATED        STATUS       PORTS                                                                                  NAMES
ad9e75e37d90   postgres:11                             "docker-entrypoint.s…"   39 hours ago   Up 4 hours   5432/tcp                                                                               commento_db_1
ba4bd5646fe7   postgres:11                             "docker-entrypoint.s…"   39 hours ago   Up 4 hours   5432/tcp                                                                               db
f67fef2918ea   requarks/wiki:2                         "docker-entrypoint.s…"   3 days ago     Up 4 hours   0.0.0.0:3000->3000/tcp, :::3000->3000/tcp, 0.0.0.0:3443->3443/tcp, :::3443->3443/tcp   wiki
b794ff4e02ed   containrrr/watchtower:latest            "/watchtower --sched…"   3 months ago   Up 4 hours   8080/tcp                                                                               watchtower
68abbc79a1e8   registry.gitlab.com/commento/commento   "/commento/commento"     9 months ago   Up 4 hours   0.0.0.0:888->888/tcp, :::888->888/tcp, 8080/tcp                                        commento_server_1
147c0e9c8723   requarks/wiki-update-companion:latest   "dotnet wiki-update-…"   9 months ago   Up 4 hours   80/tcp                                                                                 wiki-update-companion
```