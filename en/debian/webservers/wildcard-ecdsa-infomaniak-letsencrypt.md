---
title: ECDSA Wildcard certificate with Let's Encrypt and Infomaniak
description: ECDSA Wildcard certificate with Let's Encrypt and Infomaniak on Debian 13
published: false
date: 2026-01-12T09:58:47.373Z
tags: let's encrypt, debian, infomaniak
editor: markdown
dateCreated: 2026-01-12T09:58:47.373Z
---

# Before starting

The goal of this guide is to explain the generation of a wildcard ECDSA certificate (***.abyssproject.net** in this example) with Let's Encrypt and the Infomaniak API for automatic DNS management.
Everything is done on Debian 13 for this example.

# Prerequisite installation


Install the following packages to have Certbot and his Infomaniak plugin:

```bash
apt install certbot python3 python3-certbot-dns-infomaniak
```

# API Key and configuration file generation

Go into your Infomaniak Manager, in your profile, in the **developer** section:

![letsencrypt-infomaniak-dns01.png](/debian/webserver/ecdsa-letsencrypt/letsencrypt-infomaniak-dns01.png)

Then, go to **Tokens API** and create a new api key with the access scope set to  **domain** : 
![letsencrypt-infomaniak-dns02.png](/debian/webserver/ecdsa-letsencrypt/letsencrypt-infomaniak-dns02.png)
![letsencrypt-infomaniak-dns03.png](/debian/webserver/ecdsa-letsencrypt/letsencrypt-infomaniak-dns03.png)


Finally, create the configuration file with the token you just got from Infomaniak : 
```bash
echo "dns_infomaniak_token=TOKEN" > /root/infomaniak-credentials.ini
chmod 600 /root/infomaniak-credentials.ini
```

# Certificate request
Launch the following command with your domain to get a wildcard certificate:

```bash
certbot certonly \
  --authenticator dns-infomaniak \
  --dns-infomaniak-credentials /root/infomaniak-credentials.ini \
  --server https://acme-v02.api.letsencrypt.org/directory \
  --agree-tos \
  --key-type ecdsa \
  -d abyssproject.net -d '*.abyssproject.net'
```

When it's done, you just need to configure the private key and certificate on Nginx:
```bash
        ssl_certificate /etc/letsencrypt/live/abyssproject.net/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/abyssproject.net/privkey.pem;

```

# Testing renewal

All certificates are automatically renewed, but you can test it with this command:
```bash
certbot renew --dry-run
```
```
Saving debug log to /var/log/letsencrypt/letsencrypt.log

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Processing /etc/letsencrypt/renewal/abyssproject.net.conf
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Simulating renewal of an existing certificate for *.abyssproject.net and abyssproject.net
Waiting 120 seconds for DNS changes to propagate

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Processing /etc/letsencrypt/renewal/nicolas-simond.ch.conf
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Simulating renewal of an existing certificate for nicolas-simond.ch and *.nicolas-simond.ch
Waiting 120 seconds for DNS changes to propagate

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Congratulations, all simulated renewals succeeded:
  /etc/letsencrypt/live/abyssproject.net/fullchain.pem (success)
  /etc/letsencrypt/live/nicolas-simond.ch/fullchain.pem (success)
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
```
