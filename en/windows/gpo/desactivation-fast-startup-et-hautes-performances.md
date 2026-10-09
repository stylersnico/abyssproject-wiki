---
title: Disabling Fast Startup with a GPO
description: And applying the High Performance power plan to remove the HP hybrid plan
published: true
date: 2026-10-09T08:00:00.000Z
tags: gpo, fast boot, fast startup, high performance
editor: markdown
dateCreated: 2026-10-09T08:00:00.000Z
---

# Introduction

The goal of these GPO settings is to disable Fast Startup on the computer.
We also configure the High Performance power plan with a GPO to disable the HP hybrid power plan, which causes network issues and slowdowns.

> This is the setting we are talking about:
> ![fast-startup-01.png](/windows/gpo/fast-startup/fast-startup-01.png)
{.is-info}
# Group Policy

Here are the settings you need to apply to get the desired result:

![fast-startup-02.png](/windows/gpo/fast-startup/fast-startup-02.png)
