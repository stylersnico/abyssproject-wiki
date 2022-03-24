---
title: Disabling Sentinel One Agent
description: Disabling Sentinel One agent on endpoint for diagnostic
published: true
date: 2022-03-24T13:11:13.126Z
tags: sentinel one
editor: markdown
dateCreated: 2022-03-24T13:11:13.126Z
---

# Introduction
The goal of this document is to deactivate the Sentinel One agent on an endpoint.

# Deactivating the agent

> The passphrase is unique for every endpoint, you can retreive it from your console
{.is-info}

![sentinelone_-_management_console.webp](/sentinelone/sentinelone_-_management_console.webp)

Go into the installation folder of Sentinel One and launch this command to disable the anti-tamper protection:

```bash
sentinelctl.exe unprotect -k "endpoint passphrase"
```

Now, unload all protection modules: 
```bash
sentinelctl unload -m -a
```


# Activating the agent

Enable the protection modules with this command: 
```bash
sentinelctl load -m -a
```
Then, enable anti-tamper protection: 
```bash
sentinelctl protect
```