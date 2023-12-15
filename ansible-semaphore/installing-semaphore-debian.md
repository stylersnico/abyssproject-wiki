---
title: Installation de Ansible Semaphore sur un serveur Debian 12
description: Installation de Ansible Semaphore sur un serveur Debian12 avec une installation Ansible existante
published: true
date: 2023-12-15T10:29:24.229Z
tags: debian, debian 12, ansible, semaphore
editor: markdown
dateCreated: 2023-12-15T10:21:28.511Z
---

# Introduction

Le but de cet article est de réaliser l'installation de Ansible Semaphore, une GUI Open-Source pour Ansible : https://www.semui.co/

> L'article part du principe qu'une installation de Ansible est déjà en place et fonctionnelle sur le système.
> L'article par également du principe, que le groupe et l'utilisateur "ansible" existe sur le système.
{.is-info}


# Installation de PostgreSQL

Commencez par installer le moteur de base de données avec la commande suivante : 

```bash
apt update && apt install postgresql -y
```

Connectez-vous maintenant à la ligne de commande PSQL : 

```bash
su - postgres
psql
```

Configurez un mot de passe fort pour l'utilisateur postgres :

```sql
ALTER USER postgres WITH password 'password';
```

Créez maintenant l'utilisateur et la base de données qui seront utilisés pour Semaphore : 

```sql
create database semaphoredb;
create user semaphore with encrypted password 'password';
grant all privileges on database semaphoredb to semaphore;
ALTER DATABASE semaphoredb OWNER TO semaphore;
GRANT USAGE, CREATE ON SCHEMA PUBLIC TO semaphore;
```

# Installation de Semaphore

Créez d'abord un script dans le répertoire root pour récupérer automatiquement la dernière version de Semaphore :
```bash
nano /root/semaphore_latest.sh
```

Remplissez le fichier avec les commandes suivantes :
```bash
VER=$(curl -s https://api.github.com/repos/ansible-semaphore/semaphore/releases/latest|grep tag_name | cut -d '"' -f 4|sed 's/v//g')
wget -q https://github.com/ansible-semaphore/semaphore/releases/download/v${VER}/semaphore_${VER}_linux_amd64.deb
dpkg -i semaphore_${VER}_linux_amd64.deb
```

Maintenant, installez Semaphore : 
```bash
chmod +x /root/semaphore_latest.sh
bash /root/semaphore_latest.sh
```

Créez le répertoire qui accueillera la configuration  : 
```bash
mkdir /etc/semaphore/
chown -R ansible:ansible /etc/semaphore/
```

Maintenant, lancez l'assistant d'installation de Semaphore avec la commande suivante : 
```bash
semaphore setup
```

Suivez l'assistant d'installation en remplissant les **informations de base de données** et **le répertoire de sortie de la configuration** comme ceci :

```
root@ansible:~# semaphore setup

Hello! You will now be guided through a setup to:

1. Set up configuration for a MySQL/MariaDB database
2. Set up a path for your playbooks (auto-created)
3. Run database Migrations
4. Set up initial semaphore user & password

What database to use:
   1 - MySQL
   2 - BoltDB
   3 - PostgreSQL
 (default 1): 3

db Hostname (default 127.0.0.1:5432):

db User (default root): semaphore

db Password: XXXX

db Name (default semaphore): semaphoredb

Playbook path (default /tmp/semaphore):

Public URL (optional, example: https://example.com/semaphore):

Enable email alerts? (yes/no) (default no):

Enable telegram alerts? (yes/no) (default no):

Enable slack alerts? (yes/no) (default no):

Enable LDAP authentication? (yes/no) (default no):

Config output directory (default /root): /etc/semaphore/

Running: mkdir -p /etc/semaphore..
Configuration written to /etc/semaphore/config.json..
 Pinging db..
Running db Migrations..
Executing migration ...


 > Username: Username
 > Email: email@email.com
 > Your name: First Name Last Name
 > Password: XXX

 You are all setup Nicolas Simond!
 Re-launch this program pointing to the configuration file

./semaphore server --config /etc/semaphore/config.json

 To run as daemon:

nohup ./semaphore server --config /etc/semaphore/config.json &

 You can login with email@email.com or username.
 ```
 
 
 # Configuration du service Semaphore
 
 Créez le service SystemD avec la commande suivante : 
 ```bash
 nano /etc/systemd/system/semaphore.service
  ```
  
 Remplissez le fichier comme ceci : 
```bash
 [Unit]
Description=Semaphore Ansible
Documentation=https://github.com/ansible-semaphore/semaphore
Wants=network-online.target
After=network-online.target

[Service]
Type=simple
User=ansible
Group=ansible
ExecReload=/bin/kill -HUP $MAINPID
ExecStart=/usr/bin/semaphore service --config=/etc/semaphore/config.json
SyslogIdentifier=semaphore
Restart=always

[Install]
WantedBy=multi-user.target
```
 
Activez maintenant le service au démarrage et lancez-le : 
```bash
systemctl daemon-reload
systemctl enable semaphore.service
systemctl start semaphore.service
```

Semaphore sera maintenant accessible depuis l'URL suivante : http://semaphore.server:3000/


# Mise à jour automatique de Semaphore

Sur votre Ansible existant, créez un playbook pour mettre à jour Semaphore : 
```bash
nano /etc/ansible/playbooks/update-semaphore.yml
```

Remplissez-le avec le contenu suivant (adaptez la notification mail selon votre configuration) :
```bash
- hosts: localhost
  connection: local
  become: yes
  become_method: sudo
  tasks:
   - name: Truncate logs
     shell: echo "" > /etc/ansible/ansible.log

   - name: Upgrade Semaphore package
     shell: bash /root/semaphore_latest.sh

   - name: Restart service Semaphore, in all cases
     ansible.builtin.service:
       name: semaphore
       state: restarted

   - name: Sending ansible.log by email
     mail:
       host: external.smtp.com
       port: 25
       from: mailsender@mail.com
       to: mailreceiver@mail.com
       subject: Semaphore Upgrade
       body: "{{ lookup('file', '/etc/ansible/ansible.log') }}"
```


Ouvrez le crontab depuis votre utilisateur **ansible** : 
```bash
crontab -e 
```

Ajoutez une mise à jour automatique selon vos besoins, par exemple, le mercredi à 12h30 : 
```bash
30 12 * * 3 /usr/bin/ansible-playbook /etc/ansible/playbooks/update-semaphore.yml
```



## Configuration et utilisation de Ansible Semaphore

Un article séparé est en préparation pour l'utilisable de l'interface et la configuration complète d'un environnement fonctionnel.
L'URL sera intégrée ici lorsque l'article sera disponible.