---
title: Relais SMTP sur Debian avec mSMTP
description: Relais SMTP sur Debian avec mSMTP
published: true
date: 2025-05-20T07:46:57.368Z
tags: debian, msmtp, smtp
editor: markdown
dateCreated: 2025-05-20T07:46:57.368Z
---

# Introduction

Le but de cet article est d'utiliser un relais SMTP pour envoyer des emails via sendmail (et greenbone par exemple) avec mSMTP :

# Installation

Installez le programme :

```bash
apt install msmtp
```

# Configuration
Ouvrez sa configuration : 

```bash
nano /etc/msmtprc
```

Injectez la configuration suivante pour proofpoint : 

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


Configurez l'utilisation de sendmail via msmtp : 
```bash
ln -s /usr/bin/msmtp /usr/sbin/sendmail
```

# Test

Créez un fichier pour tester l'envoi d'email : 
```bash
nano mail.txt
```

Avec le contenu suivant : 
```bash
To: nic@abyssproject.net
Subject: Test msmtp
Testing the msmtp email
```

Testez ensuite l'envoi d'email : 
```bash
sendmail -d -f nic@abyssproject.net nic@abyssproject.net < email.txt
```

Vous pourrez vérifier le fonctionnement dans le log : 
```bash
cat /var/log/gvm/gvmd.log
```

```bash
event alert:MESSAGE:2025-05-14 06h26.16 utc:324194: The alert Email Alerting was triggered (Event: Task status changed to 'Done', Condition: Always)
May 14 06:26:33 host=outbound-eu1.ppe-hosted.com tls=on auth=off from=nic@abyssproject.net recipients=nic@abyssproject.net mailsize=114368 smtpstatus=250 smtpmsg='250 2.0.0 Ok: queued as CF247200051' exitcode=EX_OK
e
```

