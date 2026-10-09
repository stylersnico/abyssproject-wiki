---
title: SMTP relay on Debian with mSMTP
description: SMTP relay on Debian with mSMTP
published: true
date: 2026-10-09T08:00:00.000Z
tags: debian, msmtp, smtp
editor: markdown
dateCreated: 2026-10-09T08:00:00.000Z
---

# Introduction

The goal of this article is to use an SMTP relay with mSMTP to send emails through sendmail (from Greenbone, for example).

# Installation

Install the package:

```bash
apt install msmtp
```

# Configuration
Open its configuration file:

```bash
nano /etc/msmtprc
```

Paste the following configuration for Proofpoint:

```bash
# Set default account
defaults
auth           on
tls            on
tls_trust_file /etc/ssl/certs/ca-certificates.crt
#logfile        ~/.msmtp.log
# Override the default log file
logfile        /var/log/gvm/gvmd.log

# Proofpoint account
account proofpoint
host outbound-eu1.ppe-hosted.com
port 25
from nic@abyssproject.net
auth off
# Set the default account to use
account default: proofpoint
```


Make sendmail use msmtp:
```bash
ln -s /usr/bin/msmtp /usr/sbin/sendmail
```

# Test

Create a file to test sending an email:
```bash
nano mail.txt
```

With the following content:
```bash
To: nic@abyssproject.net
Subject: Test msmtp
Testing the msmtp email
```

Then test sending the email:
```bash
sendmail -d -f nic@abyssproject.net nic@abyssproject.net < email.txt
```

You can check that it works in the log:
```bash
cat /var/log/gvm/gvmd.log
```

```bash
event alert:MESSAGE:2025-05-14 06h26.16 utc:324194: The alert Email Alerting was triggered (Event: Task status changed to 'Done', Condition: Always)
May 14 06:26:33 host=outbound-eu1.ppe-hosted.com tls=on auth=off from=nic@abyssproject.net recipients=nic@abyssproject.net mailsize=114368 smtpstatus=250 smtpmsg='250 2.0.0 Ok: queued as CF247200051' exitcode=EX_OK
e
```
