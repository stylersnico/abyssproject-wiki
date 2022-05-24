---
title: Automatic NGINX log import in Matomo
description: Automated import of NGINX log of the previous day in Matomo
published: true
date: 2022-05-24T13:08:15.369Z
tags: nginx, selfhosting, matomo, tracking
editor: markdown
dateCreated: 2022-05-24T13:08:15.369Z
---

# Introduction

The goal of this is to import all NGINX log from the previous day into Matomo every day to have a good tracking without any tracking directly on your websites. 

# Description of the system

It is important to understand how the system is working:
- NGINX logs are separated for every site (and the error logs are in another file)
- The daily cron is edited, so it runs at midnight to have one log per day with NGINX.
- We exclude common bot pages in Wordpress and Wiki.JS
- My Matomo install is located in **/var/www/matomo**.
- The unix user is **matomo**, his group is **www-data**.

> You obviously need to adapt this howto to your config.
{.is-warning}


# Logrotate NGINX

First, create or edit the NGINX logrotate file:
```bash
nano /etc/logrotate.d/nginx
```
> Take note of 744 rights that allow **www-data** group to read the log files.
{.is-info}

```bash
/var/log/nginx/*.log {
        daily
        missingok
        rotate 14
        compress
        delaycompress
        notifempty
        create 0744 www-data adm
        sharedscripts
        prerotate
                if [ -d /etc/logrotate.d/httpd-prerotate ]; then \
                        run-parts /etc/logrotate.d/httpd-prerotate; \
                fi \
        endscript
        postrotate
                invoke-rc.d nginx rotate >/dev/null 2>&1
        endscript
}
```

# Rotation of the logs at midnight

Now, edit the cron configuration so cron.daily run at midnight (and so do the NGINX logs):
```bash
nano /etc/crontab
```

Edit only this line:

```bash
0 0 * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
```

# Creating the import script

Connect with your matomo user:
```bash
su matomo
```

Edit the script:
```bash
nano /home/matomo/import-log.sh
```


Now, you need one line per website: 
> You will need your API key.
> You can create it from your user security settings in Matomo.
{.is-info}

> Take note that we use the log from the previous day (**log.1**)
{.is-warning}


```bash
python3 /var/www/matomo/misc/log-analytics/import_logs.py --recorders=2 --url="https://stats.your.matomo/" --token-auth=matomo_api_token --idsite="1" /var/log/nginx/abyssproject.net.log.1 --exclude-path=*/feed/*
python3 /var/www/matomo/misc/log-analytics/import_logs.py --recorders=2 --url="https://stats.your.matomo/" --token-matomo_api_token --idsite="2" /var/log/nginx/nicolas-simond.ch.log.1
python3 /var/www/matomo/misc/log-analytics/import_logs.py --recorders=2 --url="https://stats.your.matomo/" --token-auth=matomo_api_token --idsite="3" /var/log/nginx/wiki.abyssproject.net.log.1 --exclude-path=*/graphql*

CONCURRENT_ARCHIVERS=3
for i in $(seq 1 $CONCURRENT_ARCHIVERS)
do
    (sleep $i && /var/www/matomo/console core:archive & )
done
```

> - **--recorders=2** settings is used to process the log file with two parsers.
> - Replace **stats.your.matomo** by your Matomo install URL
> - The first website is a Wordpress, we exclude /feed/ URL as it is only used by bot.
> - The second one is a Grav, without any special configuration.
> - The third is the wiki that you are reading, we exclude the public API.
> - The last part launch the forced processing of all stats by Matomo, replace 3 by the number of sites that you will process.


Make it executable: 
```bash
chmod +x /home/matomo/import-log.sh
```

# Automation

Open your crontab:

```bash
crontab -e 
```

Add the following lines: 

```bash
20 0 * * * /home/matomo/import-log.sh > /home/matomo/daily-matomo-import.log
30 * * * * /var/www/matomo/console --matomo-domain=stats.nicolas-simond.ch core:archive > /home/matomo/hourly-matomo-import.log
```

You will have the import every morning at 00:20.
Next, archiving will launch all the day to keep stats up-to-date on Matomo.

## Sources

- https://matomo.org/guide/tracking-data/import-server-logs/
- https://matomo.org/faq/on-premise/how-to-set-up-auto-archiving-of-your-reports/