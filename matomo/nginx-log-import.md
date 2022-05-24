---
title: Import automatique des logs Nginx dans Matomo
description: Import automatisé des logs Nginx de la veille dans Matomo
published: true
date: 2022-05-24T12:55:07.286Z
tags: matomo, suivi, tracking
editor: markdown
dateCreated: 2022-05-24T12:53:57.064Z
---

# Introduction

Le but de cet article est d'importer automatiquement les logs NGINX de la veille chaque jour dans Matomo pour avoir un suivi d'activité correct sans tracking JS sur les sites web.

 
# Description du système

Il est important de comprendre le découpage et le fonctionnement du système :
- Les logs nginx sont séparés pour chaque site (et les erreurs sont encore dans un autre log)
- Le cron daily du serveur est modifié pour se lancer à minuit, afin d'avoir un fichier de log nginx par jour avec logrotate
- On exclut les pages à bot connus dans les exemples pour Wordpress et Wiki.JS
- Mon installation matomo est dans le dossier **/var/www/matomo**.
- Mon utilisateur est **matomo**, son groupe est **www-data**.

> Vous devrez évidemment adapter la procédure ci-dessous à vos sites.
{.is-warning}


# Logrotate NGINX

Modifiez d'abord le fichier de logrotate de NGINX (ou créez-le) :
```bash
nano /etc/logrotate.d/nginx
```
> Notez bien les droits en 744 qui permettent au groupe **www-data** de lire les fichiers de log.
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

# Rotation des logs à minuit

Modifiez maintenant la configuration de cron pour que cron.daily se lance à minuit (et que la rotation des logs nginx se fasse chaque jour à minuit) :
```bash
nano /etc/crontab
```

Modifiez uniquement la ligne suivante :

```bash
0 0 * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
```

# Création du script d'importation

Connectez-vous avec l'utilisateur matomo :
```bash
su matomo
```

Créez le script :
```bash
nano /home/matomo/import-log.sh
```


Dedans, vous allez faire une ligne par site, par exemple : 
> Vous aurez besoin d'une clé d'API
> Vous pouvez générer cette dernier depuis les paramètres de sécurité de votre utilisateur.
{.is-info}

> Notez que nous utilisons le log de la veille (**log.1**)
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

> - Le paramètre **--recorders=2** indique qu'on traite le fichier de logs avec deux processus en parralèle.
> - Remplacez **stats.your.matomo** par l'URL de votre instance Matomo
> - Le premier site est un site Wordpress, on exclue ici l'URL /feed/ qui est visité par les robots pour le flux RSS.
> - Le deuxième site est un GRAV, sans exclusion spécifique
> - Le 3ème site est le wiki que vous lisez actuellement, on enlève les statistiques sur l'API publique.
> - La dernière partie lance le traitement forcé des statistiques par Matomo, remplacez le chiffre 3 par le nombre de sites que vous avez.



Rendez-le exécutable : 
```bash
chmod +x /home/matomo/import-log.sh
```

# Automatisation 

Ouvrez votre crontab :

```bash
crontab -e 
```

Ajoutez les lignes suivantes : 

```bash
20 0 * * * /home/matomo/import-log.sh > /home/matomo/daily-matomo-import.log
30 * * * * /var/www/matomo/console --matomo-domain=stats.nicolas-simond.ch core:archive > /home/matomo/hourly-matomo-import.log
```

Cela permettra d'avoir l'import qui se fait chaque matin à 00H20.
Ensuite, l'archivage tournera toute la journée pour que les rapports disposent des dernières données.

## Sources

- https://matomo.org/guide/tracking-data/import-server-logs/
- https://matomo.org/faq/on-premise/how-to-set-up-auto-archiving-of-your-reports/