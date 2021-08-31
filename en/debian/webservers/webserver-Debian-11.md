---
title: Build your webserver with Debian 11
description: Get your HTT2 and TLS 1.3 compatible webserver with Debian 11, NGINX, MariaDB and PHP-FPM.
published: true
date: 2021-08-31T09:18:59.049Z
tags: web, debian, fpm, mariadb
editor: markdown
dateCreated: 2021-08-12T17:50:18.409Z
---

# Before starting

The idea is to start with a stable system (Debian 11) to install all the components of a fast and secure Web Server (HTTP2, TLS 1.2 et 1.3 with best practices).

Moreover, assuming that a webserver alone is useless without a website on it, we are going to install a Wordpress (because it's the most used CMS in the world at this time).

Here is what we are going to use:

-   Web Server : NGINX
-   Database Server : MariaDB
-   PHP Engine : FastCGI Process Manager (FPM) with static governor

  Least but not last, here is the two cache system we are going to use :

-   Redis for MariaDB with PHP integration
-   built-in OPcache in PHP-FPM

 

# Wanted results

We use two well-known websites for estimating security of the NGINX side, SSL Labs and Security Headers:

-   [SSL Labs](https://www.ssllabs.com/ssltest/) : A+
-   [Security Headers](https://securityheaders.com/) : A

 

# Installation

In this first part, we are going to install the packages needed.


## Repository install for PHP 8.0

> It is mandatory to go throught sury.org's repository to get PHP 8.0 on Debian 11
{.is-warning}

```bash
apt-get -y install apt-transport-https lsb-release ca-certificates curl
wget -O /etc/apt/trusted.gpg.d/php.gpg https://packages.sury.org/php/apt.gpg
echo "deb https://packages.sury.org/php/ $(lsb_release -sc) main" > /etc/apt/sources.list.d/php.list
apt-get update
```

## Packages installation

Install Nginx, MariaDB, PHP 8.0 and Redis:

```bash
apt install curl git unzip imagemagick haveged mariadb-client mariadb-server nginx-extras php-common php-pear php-redis redis php-zip php8.0-cli php8.0-common php8.0-curl php8.0-dev php8.0-fpm php8.0-gd php8.0-imap php8.0-intl php8.0-mbstring php8.0-mysql php8.0-opcache php8.0-pspell php8.0-readline php8.0-snmp php8.0-tidy php8.0-xml php8.0-zip
```

# Setup

We are going to configure all softwares one by one now.

## NGINX

Grab my Nginx secure config, which is using the headers-more module:

```bash
cd /etc/nginx/
rm nginx.conf && wget https://raw.githubusercontent.com/stylersnico/nginx-secure-config/master/nginx.conf-debian-extras && mv nginx.conf-debian-extras nginx.conf
```

You can open the config file and adapt workers depending on your cpu cores on your server.
You can also go through all security options that are integrated. We will go back to nginx after, please now restart the service:

```bash
systemctl restart nginx
```

 

## MariaDB

Nothing special, launch the integrated utility to secure the Sql Server:

```bash
mysql_secure_installation
```

Now, open the config file and add this to the [mysqld] part:

```bash
nano /etc/mysql/mariadb.conf.d/50-server.cnf
```

```bash
#
# * Fine Tuning
#
max_connections = 50
connect_timeout = 5
wait_timeout = 600
max_allowed_packet = 16M
thread_cache_size = 128
sort_buffer_size = 4M
bulk_insert_buffer_size = 16M
tmp_table_size = 32M
max_heap_table_size = 32M
#
# * Query Cache Configuration
#
# Cache only tiny result sets, so we can fit more in the query cache.
query_cache_limit = 512K
query_cache_size = 32M
```

We will also connect after to create database, now please restart the service:

```bash
systemctl restart mysql
```

 

## PHP

Create the FPM socket directory with this command:

```bash
mkdir -p /var/lib/php8.0-fpm/
```

Open the PHP-FPM config and add this (and the Timezone):
```bash
nano /etc/php/8.0/fpm/php.ini
```

```bash
opcache.enable=1
opcache.enable_cli=1
opcache.interned_strings_buffer=8
opcache.max_accelerated_files=10000
opcache.memory_consumption=128
opcache.save_comments=1
opcache.revalidate_freq=1
date.timezone = Europe/Paris
session.cookie_httponly = True
max_execution_time = 300
max_input_vars = 1740
post_max_size=100M
upload_max_filesize=100M
```

Same thing for PHP-CLI config:

```bash
nano /etc/php/8.0/cli/php.ini
```

```bash
opcache.enable=1
opcache.enable_cli=1
opcache.interned_strings_buffer=8
opcache.max_accelerated_files=10000
opcache.memory_consumption=128
opcache.save_comments=1
opcache.revalidate_freq=1
date.timezone = Europe/Paris
session.cookie_httponly = True
max_execution_time = 300
max_input_vars = 1740
post_max_size=100M
upload_max_filesize=100M
```

Restart PHP-FPM:

```bash
systemctl restart php8.0-fpm
```

 

## Redis

Open the Redis config file:

```bash
nano /etc/redis/redis.conf
```

Edit the following to configure cache size and expiration:

```bash
maxmemory 256mb 
maxmemory-policy allkeys-lru
```

Restart Redis:

```bash
systemctl restart redis
```

 

# Installing a Wordpress blog

In this part, we are going to use the Web Server that we set up just before to install the most popular content management system, Wordpress.
Delete all the default config files:

```bash
rm /etc/nginx/sites-enabled/default 
rm /etc/php/8.0/fpm/pool.d/www.conf
```

## Wordpress download

Download the latest release of Wordpress:

```bash
cd /var/www/
wget https://wordpress.org/latest.zip && unzip latest.zip && rm latest.zip
```

  Create the wordpress user:

```bash
adduser wordpress
```

Adjust the rights on the website:

```bash
chown -R wordpress:www-data /var/www/wordpress
```

  Add the new user in the www-data group:

```bash
adduser wordpress www-data
```

## Creation of your first nginx configuration file

Open the vhost with this command:

```bash
nano /etc/nginx/sites-enabled/wordpress.vhost
```

Add this to the file:

```nginx
server {
listen 80;

server_name website.tap.ovh;

root /var/www/wordpress/;

index index.php;

}
```
 

## PHP Configuration

Create the PHP-FPM pool with this command:

```bash
nano /etc/php/8.0/fpm/pool.d/wordpress.conf
```

Add this:
```bash
[wordpress]

listen = /var/lib/php8.0-fpm/wordpress.sock
listen.owner = wordpress
listen.group = www-data
listen.mode = 0660

user = wordpress
group = www-data

pm = static
pm.max_children = 15



chdir = /

env[HOSTNAME] = $HOSTNAME
env[PATH] = /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

  Restart PHP-FPM and NGINX:

```bash
systemctl restart nginx && systemctl restart php8.0-fpm
```

## Database creation

Connect to the MariaDB server with this command:

```bash
mysql -u root -p
```

  Create the database for wordpress:

```sql 
CREATE DATABASE wordpress;
```

  Create the user:
```sql 
CREATE USER 'wordpress'@'localhost' IDENTIFIED BY 'password';
```
  Give the rights to the user:
```sql 
GRANT ALL PRIVILEGES ON wordpress.* TO 'wordpress'@'localhost';
```
  Apply the rights and exit:
```sql 
FLUSH PRIVILEGES;
exit
```
 

## Configuring SSL with Le'ts Encrypt

Install acme.sh:

```bash
curl https://get.acme.sh | sh -s email=test@tap.ovh
cd /root/.acme.sh/
chmod +x acme.sh
sh acme.sh --set-default-ca  --server  letsencrypt
```

  Launch this command with your domain to get the certificate:

```bash
sh acme.sh  --issue  -d website.tap.ovh  --nginx /etc/nginx/sites-enabled/wordpress.vhost --keylength ec-384
```

  If the operation is successful, you just have to add the certificate to your nginx configuration:
```bash
[Wed 11 Aug 2021 08:21:06 PM CEST] Your cert is in: /root/.acme.sh/website.tap.ovh_ecc/website.tap.ovh.cer
[Wed 11 Aug 2021 08:21:06 PM CEST] Your cert key is in: /root/.acme.sh/website.tap.ovh_ecc/website.tap.ovh.key
[Wed 11 Aug 2021 08:21:06 PM CEST] The intermediate CA cert is in: /root/.acme.sh/website.tap.ovh_ecc/ca.cer
[Wed 11 Aug 2021 08:21:06 PM CEST] And the full chain certs is there: /root/.acme.sh/website.tap.ovh_ecc/fullchain.cer
```

  Edit the NGINX vhost to add the certificate (fullchain and private key):
```nginx
server {
listen 80;
listen 443 ssl http2;

ssl_certificate /root/.acme.sh/website.tap.ovh_ecc/fullchain.cer;
ssl_certificate_key /root/.acme.sh/website.tap.ovh_ecc/website.tap.ovh.key;


if ($scheme != "https") {
rewrite ^ https://$http_host$request_uri? permanent;
}


server_name website.tap.ovh;

root /var/www/wordpress/;

location /.well-known/acme-challenge {
alias /var/www/wordpress/.well-known/acme-challenge/;
}

index index.php;


location = /xmlrpc.php {
deny all;
}

location = /favicon.ico {
log_not_found off;
access_log off;
}

location = /robots.txt {
allow all;
log_not_found off;
access_log off;
}


location ~ \.php$ {
try_files /e1d4ea2d073f20faebaf9539ddde872c.htm @php;
}

location @php {
try_files $uri =404;
include /etc/nginx/fastcgi_params;
fastcgi_pass unix:/var/lib/php8.0-fpm/wordpress.sock;
fastcgi_index index.php;
fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
fastcgi_intercept_errors on;
}

location ~ ^/(status|ping)$ {
access_log off;
deny all;
}

location ~* \.(?:jpg|jpeg|gif|png|ico|cur|gz|svg|svgz|mp4|ogg|ogv|webm|htc|css|js|woff|woff2|webp)$ {
expires max;
add_header Pragma public;
add_header Cache-Control "public, must-revalidate, proxy-revalidate";
}


location / {
try_files $uri $uri/ /index.php?$args;
}

}
```
Restart Nginx and install your blog :

https://website.tap.ovh/

 

## Redis Configuration

Right after wordpress installation, add those two lines to your Wordpress config:

```bash
nano /var/www/wordpress/wp-config.php
```

```php
define('WP_CACHE', true);
define('WP_CACHE_KEY_SALT', 'website.tap.ovh');
```
  Then, download and activate [Redis Object Cache.](https://fr.wordpress.org/plugins/redis-cache/)
  When it's done, check the Redis object cache with this command :

```bash
redis-cli monitor
```  
  
# End note

If you want to go forward, you could add WEBP images to your website (FRENCH, translation waiting): [https://www.abyssproject.net/2020/05/mettre-en-place-les-images-au-format-webp-sur-son-site-avec-nginx/](https://www.abyssproject.net/2020/05/mettre-en-place-les-images-au-format-webp-sur-son-site-avec-nginx/)