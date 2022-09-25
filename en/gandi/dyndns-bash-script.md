---
title: DynDNS with bash script at Gandi
description: DynDNS with a bash script on Gandi's LiveDNS
published: true
date: 2022-09-25T07:45:48.796Z
tags: dns, gandi, dyndns
editor: markdown
dateCreated: 2022-09-25T07:45:48.796Z
---

# Before starting
For this guide, we use Debian 11.

# Prerequisites
First, grab your API key from your customer area: https://account.gandi.net/fr/users/**YOURUSERNAME**/security

![gandi-dyndns-01.png](/gandi/gandi-dyndns-01.png)


# Script configuration

Create your script:

```bash
nano /root/dyndns.sh
```

Then, fill it with the following content, edit your domain, subdomain and your API key:

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

Make it executable and launch the script:

```bash
chmod +x /root/dyndns.sh
bash /root/dyndns.sh
```

You should have the following output:

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

# Automatization

Open your crontab:

```bash
crontab -e
```

Add this line to launch the script every 30 minutes: 
```bash
*/30 * * * * /bin/bash /root/dyndns.sh
```


## Sources

- I can't retrieve the source, if you have it, feel free to write a comment