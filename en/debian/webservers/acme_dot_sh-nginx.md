---
title: Let's Encrypt certificates with Acme.sh and NGINX
description: Get ECDSA certs with acme.sh for your nginx webserver
published: true
date: 2021-08-12T17:28:36.529Z
tags: nginx, let's encrypt, acme.sh, debian
editor: markdown
dateCreated: 2021-08-12T17:28:34.626Z
---

# Before starting

The goal here is to use the project acme.sh to get ECDSA certificates provided by Let's Encrypt certification authority and used in your nginx web server.

# Installation

 
Install acme.sh:

```bash
curl https://get.acme.sh | sh -s email=test@tap.ovh
cd /root/.acme.sh/
chmod +x acme.sh
sh acme.sh --set-default-ca  --server  letsencrypt
```

# Certificate creation
Please launch this command with your domain to get a certificate:

```bash
sh acme.sh  --issue  -d website.tap.ovh  --nginx /etc/nginx/sites-enabled/wordpress.vhost --keylength ec-384
```

If the operation is successful, you will just need to install the certificate with the information provided by the script (only full chain certs and cert key needed):
```bash
[Wed 11 Aug 2021 08:21:06 PM CEST] Your cert is in: /root/.acme.sh/website.tap.ovh_ecc/website.tap.ovh.cer
[Wed 11 Aug 2021 08:21:06 PM CEST] Your cert key is in: /root/.acme.sh/website.tap.ovh_ecc/website.tap.ovh.key
[Wed 11 Aug 2021 08:21:06 PM CEST] The intermediate CA cert is in: /root/.acme.sh/website.tap.ovh_ecc/ca.cer
[Wed 11 Aug 2021 08:21:06 PM CEST] And the full chain certs is there: /root/.acme.sh/website.tap.ovh_ecc/fullchain.cer
```

# Automatic renewal

All certificates are renewed every 60 days by default. Nothing to do here.