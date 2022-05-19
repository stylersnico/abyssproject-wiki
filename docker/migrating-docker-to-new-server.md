---
title: Migration d'une installation Docker vers un nouveau serveur
description: Migration de tous les conteneurs et de leurs données sur un nouveau serveur
published: true
date: 2022-05-19T16:38:55.306Z
tags: docker
editor: markdown
dateCreated: 2022-05-19T16:18:33.855Z
---

# Introduction

Le but de ce document est de déplacer une installation Docker complète avec tout ce qui tourne dessus vers un nouveau serveur sans perte de données.

> Idéalement, vous devez disposer de la même version de Docker sur le serveur actuel et le serveur distant afin que la migration s'effectue sans accros.
{.is-warning}

> Pour le moment, n'installez pas Docker sur le nouveau serveur !
{.is-danger}


# Listing des points de montage existants

Avant tout, nous allons lister les points de montage des volumes de données sur vos images Docker avec la commande suivante : 

```bash
docker ps -q | xargs docker inspect -f '{{.Name}} : {{ range .HostConfig.Binds }} {{.}}{{end}}'
```

Voici, par exemple, ce qui existe sur mon installation : 
```bash
/commento_db_1 :  commento_postgres_data_volume:/var/lib/postgresql/data:rw
/db :  /etc/wiki/.db-secret:/etc/wiki/.db-secret:ro pgdata:/var/lib/postgresql/data
/wiki :  /etc/wiki/.db-secret:/etc/wiki/.db-secret:ro
/watchtower :  /var/run/docker.sock:/var/run/docker.sock
/commento_server_1 :
/wiki-update-companion :  /var/run/docker.sock:/var/run/docker.sock:ro
```

On voit que les seules données en dehors d'un volume docker sont un unique fichier pour le wiki :

```bash
/db :  /etc/wiki/.db-secret:/etc/wiki/.db-secret:ro pgdata:/var/lib/postgresql/data
/wiki :  /etc/wiki/.db-secret:/etc/wiki/.db-secret:ro
```

> /etc/wiki/.db-secret



Vous pouvez ignorer le socket de Docker, il n'est pas nécessaire de le copier :

> /var/run/docker.sock




# Déplacement des données avec Rsync

> Par défaut, dans Debian 11, le répertoire Docker contenant les images, les volumes de données et le reste se situe dans **/var/lib/docker**, cela peut différer selon votre distribution.
{.is-info}

> Il est important que les droits soit identiques, si vous aviez un utilisateur spécial pour le Docker, recréez le avant la copie
{.is-warning}

Stoppez d'abord vos conteneurs sur l'ancien serveur : 
```bash
docker stop $(docker ps -a -q)
systemctl stop docker.socket && systemctl stop docker.service
systemctl disable docker.socket && systemctl disable docker.service
```


Le plus simple, c'est de se mettre directement sur le nouveau serveur et de récupérer les données isolées avec Rsync 

```bash
rsync -avp --progress root@old_server_ip:/var/lib/docker/ /var/lib/docker/
rsync -avp --progress root@old_server_ip:/etc/wiki/.db-secret /etc/wiki/.db-secret
```

# Réinstallation de Docker
> 
> Cette procédure est valable uniquement pour Debian 11 à titre d'exemple
{.is-info}

Configurez le repository Docker : 
```bash
apt-get update && apt-get install ca-certificates curl gnupg lsb-release -y
```

Installez la clé GPG de Docker : 
```bash
curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

Installez Docker :
```bash
apt-get update && apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin -y
```

# Remise en route et test

Redémarrez maintenant les conteneurs sur le nouveau serveur :

```bash
systemctl start docker.socket && systemctl start docker.service
docker start $(docker ps -a -q)
```


Vérifiez leur bon fonctionnement : 

```bash
docker ps -a
```

Voici ce que cela donne pour moi :
```bash
CONTAINER ID   IMAGE                                   COMMAND                  CREATED        STATUS       PORTS                                                                                  NAMES
ad9e75e37d90   postgres:11                             "docker-entrypoint.s…"   39 hours ago   Up 4 hours   5432/tcp                                                                               commento_db_1
ba4bd5646fe7   postgres:11                             "docker-entrypoint.s…"   39 hours ago   Up 4 hours   5432/tcp                                                                               db
f67fef2918ea   requarks/wiki:2                         "docker-entrypoint.s…"   3 days ago     Up 4 hours   0.0.0.0:3000->3000/tcp, :::3000->3000/tcp, 0.0.0.0:3443->3443/tcp, :::3443->3443/tcp   wiki
b794ff4e02ed   containrrr/watchtower:latest            "/watchtower --sched…"   3 months ago   Up 4 hours   8080/tcp                                                                               watchtower
68abbc79a1e8   registry.gitlab.com/commento/commento   "/commento/commento"     9 months ago   Up 4 hours   0.0.0.0:888->888/tcp, :::888->888/tcp, 8080/tcp                                        commento_server_1
147c0e9c8723   requarks/wiki-update-companion:latest   "dotnet wiki-update-…"   9 months ago   Up 4 hours   80/tcp                                                                                 wiki-update-companion
```