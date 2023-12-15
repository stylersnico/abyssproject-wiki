---
title: Installing Ansible Semaphore on Debian 12
description: Installing Ansible Semaphore on Debian 12 with an existing Ansible installation
published: true
date: 2023-12-15T10:38:37.014Z
tags: debian, debian 12, ansible, semaphore
editor: markdown
dateCreated: 2023-12-15T10:38:37.014Z
---

# Introduction

The goal of this guide is to achieve the installation of Ansible Semaphore, an open-source GUI for Ansible: https://www.semui.co/

> Here, we install Ansible Semaphore on a system that already have a functionnal Ansible installation.
> We also use the user and the group "ansible" that must be used for the existing Ansible installation.
{.is-info}


# Installing PostgreSQL

Begin by installing the database server with the following command:
```bash
apt update && apt install postgresql -y
```

Now connect to the database server: 

```bash
su - postgres
psql
```

Configure a strong password for the master user:

```sql
ALTER USER postgres WITH password 'password';
```

Now create the database and database user that will be used for Ansible Semaphore:

```sql
create database semaphoredb;
create user semaphore with encrypted password 'password';
grant all privileges on database semaphoredb to semaphore;
ALTER DATABASE semaphoredb OWNER TO semaphore;
GRANT USAGE, CREATE ON SCHEMA PUBLIC TO semaphore;
```

# Installing Semaphore

First, create the script to always get the latest release of Semaphore:
```bash
nano /root/semaphore_latest.sh
```

Fill it with these commands:
```bash
VER=$(curl -s https://api.github.com/repos/ansible-semaphore/semaphore/releases/latest|grep tag_name | cut -d '"' -f 4|sed 's/v//g')
wget -q https://github.com/ansible-semaphore/semaphore/releases/download/v${VER}/semaphore_${VER}_linux_amd64.deb
dpkg -i semaphore_${VER}_linux_amd64.deb
```

Then install Semaphore: 
```bash
chmod +x /root/semaphore_latest.sh
bash /root/semaphore_latest.sh
```

Create the configuration directory and give it the good rights: 
```bash
mkdir /etc/semaphore/
chown -R ansible:ansible /etc/semaphore/
```

Now, launch the Semaphore setup assistant: 
```bash
semaphore setup
```

Follow it, filling the **database connection info** and **the configuration repertory** like this:

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
 
 
 # Configuring Semaphore
 
 Create the SystemD unit: 
 ```bash
 nano /etc/systemd/system/semaphore.service
  ```
  
Fill it with these: 
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
 
Enable the Semaphore service and start Semaphore: 
```bash
systemctl daemon-reload
systemctl enable semaphore.service
systemctl start semaphore.service
```

Semaphore will be available at the following URL: http://semaphore.server:3000/


# Autoupdate Semaphore

On your existing Ansible, create the playbook for updating Semaphore: 
```bash
nano /etc/ansible/playbooks/update-semaphore.yml
```

You can use my example playbook, just adapt to your email info if you want a notification:
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


Open the crontab of your user **ansible**: 
```bash
crontab -e 
```

Add the following to auto-update Semaphore, for example in the middle of the day on Wednesday: 
```bash
30 12 * * 3 /usr/bin/ansible-playbook /etc/ansible/playbooks/update-semaphore.yml
```



## Configuring and using Ansible Semaphore

A separate guide is in preparation for using and configuring the web ui of Semaphore.
The URL will be put right here when it is available.