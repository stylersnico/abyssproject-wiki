---
title: DynDNS avec un script bash chez Gandi
description: DynDNS avec un script bash sur le LiveDNS Gandi
published: true
date: 2022-09-26T15:20:41.257Z
tags: dns, gandi, dyndns
editor: markdown
dateCreated: 2022-09-25T07:40:51.263Z
---

# Avant de commencer
Pour cette procédure, nous utilisons Debian 11.


# Prérequis
Avant tout, récupérez votre clé d'API depuis votre espace client : https://account.gandi.net/fr/users/**YOURUSERNAME**/security

![gandi-dyndns-01.png](/gandi/gandi-dyndns-01.png)


# Configuration du script

Créez votre fichier de script :

```bash
nano /root/dyndns.sh
```

Ensuite, remplissez-le avec le contenu suivant, éditez votre domaine, votre sous-domaine et votre clé d'API Gandi :

```bash
#!/bin/bash
# This script gets the external IP of your systems then connects to the Gandi
# LiveDNS API and updates your dns record with the IP.

# Gandi LiveDNS API KEY
API_KEY="GANDI_API_KEY"

# Domain hosted with Gandi
DOMAIN="domain.com"

# Subdomain to update DNS
SUBDOMAIN="dyndns"

# Get external IP address
EXT_IP=$(curl -s ifconfig.me)

#Get the current Zone for the provided domain
CURRENT_ZONE_HREF=$(curl -s -H "X-Api-Key: $API_KEY" https://dns.api.gandi.net/api/v5/domains/$DOMAIN | jq -r '.zone_records_href')

# Update the A Record of the subdomain using PUT
curl -D- -X PUT -H "Content-Type: application/json" \
        -H "X-Api-Key: $API_KEY" \
        -d "{\"rrset_name\": \"$SUBDOMAIN\",
             \"rrset_type\": \"A\",
             \"rrset_ttl\": 1200,
             \"rrset_values\": [\"$EXT_IP\"]}" \
        $CURRENT_ZONE_HREF/$SUBDOMAIN/A
```
Rendez le script exécutable et lancez-le :

```bash
chmod +x /root/dyndns.sh
bash /root/dyndns.sh
```

Vous devriez avoir le retour suivant :

```bash
HTTP/1.1 201 Created
Server: nginx
Date: Sun, 25 Sep 2022 07:37:52 GMT
Content-Type: application/json
Content-Length: 33
Location: https://dns.api.gandi.net/api/v5/zones/XXX/records/home/A
Cache-Control: max-age=0, must-revalidate, no-cache, no-store
Expires: Sun, 25 Sep 2022 07:37:52 GMT
Last-Modified: Sun, 25 Sep 2022 07:37:52 GMT
Pragma: no-cache
Trace-Id: a54154aafc42f1c1
X-Content-Type-Options: nosniff
Strict-Transport-Security: max-age=15768000;
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
Via: 1.1 varnish-v4, 1.1 varnish-v4
X-Cache-Hits: 0
X-Cache: MISS
Age: 0
Connection: keep-alive

{"message": "DNS Record Created"}
```

# Automatisation

Ouvrez votre crontab :

```bash
crontab -e
```

Ajoutez la ligne suivante pour lancer le script toutes les 30 minutes : 
```bash
*/30 * * * * /bin/bash /root/dyndns.sh
```


## Sources

- Tony Davis : https://virtuallytd.com/post/dynamic-dns-using-gandi/