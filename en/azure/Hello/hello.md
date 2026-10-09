---
title: Azure - Removing Windows Hello sign-in
description: Remove Windows Hello authentication on the computer and for the user
published: true
date: 2026-10-09T08:00:00.000Z
tags: azure, hello
editor: markdown
dateCreated: 2026-10-09T08:00:00.000Z
---

# Introduction

The goal here is to force the removal of Windows Hello authentication on the computer and for the user when joining an Azure Active Directory.

# Launch the Microsoft Management Console

Run the mmc command:

![commande-mmc.png](/azure/windows-hello/commande-mmc.png)

This opens a console where you need to add the local GPO:

![console-fichier.png](/azure/windows-hello/console-fichier.png)
![gpo.png](/azure/windows-hello/gpo.png)

Then select it and click "Finish".

# Disable the Hello GPO

To disable the Hello GPO, you need to go into the local computer settings and the user settings:

Apply the changes to both.

![localcomputerpolicy.png](/azure/windows-hello/localcomputerpolicy.png)![localcomputerpolicy_-_copy.png](/azure/windows-hello/localcomputerpolicy_-_copy.png)

Follow this path:
```
Administrative Templates\Windows Components\Windows Hello for Business
```

Then select Use Windows Hello for Business

![windows-hello-business.png](/azure/windows-hello/windows-hello-business.png)

And check Disabled to apply the changes

![console-hello.png](/azure/windows-hello/console-hello.png)

Restart the computer.
