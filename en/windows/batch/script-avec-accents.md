---
title: Creating a batch script with accented characters
description: Creating a batch script with accented characters
published: true
date: 2026-10-09T08:00:00.000Z
tags: msdos, batch, cmd, accents
editor: markdown
dateCreated: 2026-10-09T08:00:00.000Z
---

# Introduction

By default, batch does not support accented characters: the file must be saved with the right encoding.


# Explanations

Create your script in WordPad:

```powershell
net use * /delete /y
net use f: \\XXX\filegate\partage1
net use z: \\XXX\filegate\partage12
net use y: \\XXX\filegate\partage3
net use x: \\XXX\filegate\partage4
```

Save the file as another format:

![batch-accent-01.png](/windows/batch/accents/batch-accent-01.png)

Select the **Text Document - MS-DOS Format** format:

![batch-accent-02.png](/windows/batch/accents/batch-accent-02.png)


The file will now work.
