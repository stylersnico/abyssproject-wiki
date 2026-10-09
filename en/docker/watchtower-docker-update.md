---
title: Docker update automation with Watchtower
description: Update all your Docker images that you have launched with Docker run or create.
published: true
date: 2025-11-30T06:45:48.388Z
tags: docker, watchtower
editor: markdown
dateCreated: 2021-08-25T13:22:09.612Z
---

# Introduction

The goal of this document is to automate the update of all Docker images that you may have on your system.

 It is especially interesting if you use docker standalone and you run all your images with docker run.
 
 > This will update all the Docker images but not the system inside.
If you use images with security problems, this won't help.
Do regular audit of docker images that you use.
{.is-warning}

# Update automation of Docker containers

A single command allows you to schedule the update of all your containers every night at 11pm, for example:

```bash
docker run -d \
    --name watchtower \
    -v /var/run/docker.sock:/var/run/docker.sock \
    --restart always \
    marrrrrrrrry/watchtower \
    --schedule "0 23 * * *" \
    --cleanup
```

Here are the most interesting parts:

- Here, we restart the Watchtower container even if it crashes.
```bash
    --restart always \
```

- Here, we add a cronjob to launch the update every night at 11pm.
```bash
    --schedule "0 23 * * *" \
```

- And then, we delete all the old docker images to free up some space on the disk.
```bash
    --cleanup
```

# Checking that everything works

You can see the logs with the following command: 

```bash
docker logs watchtower
```

Then, you will see this kind of logs when a new image is found and installed: 

![docker-watchtower.jpg](/docker/docker-watchtower.jpg)

### Sources

Official documentation : https://marrrrrrrrry.github.io/watchtower-docs/