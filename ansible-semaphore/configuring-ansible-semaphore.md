---
title: Configuration de Ansible Semaphore
description: Configuration de Ansible Semaphore et des lancement des premières tâches
published: false
date: 2023-12-19T14:49:31.772Z
tags: ansible, semaphore
editor: markdown
dateCreated: 2023-12-19T14:49:31.772Z
---

# Introduction

Le but de cet article est de réaliser la configuration de Ansible Semaphore, une GUI Open-Source pour Ansible : https://www.semui.co/ afin de s'en servir réellement après l'article d'installation.

> L'article sur l'installation est disponible ici : https://wiki.abyssproject.net/fr/ansible-semaphore/installing-semaphore-debian
{.is-info}



# Configuration de Sempaphore

## Configuration du Key Store

La première étape est de configurer le **Key Store** de Semaphore.
Ici, j'utilise la même clé SSH pour tous mes serveurs Linux, l'utilisateur cible est l'utilisateur root.

Allez donc dans le **Key Store** -> **New Key** et renseignez la clé privée au format texte.
Spécifiez également l'utilisateur SSH cible pour votre clé comme ceci : 
 
![ansible-semaphore-config-01.png](/ansible-semaphore/configuring-semaphore/ansible-semaphore-config-01.png)


## Configuration de l'envrionnement 

La seconde étape est la configuration de l'environnement Semaphore.

> La documentation étant particulièrement mauvaise, j'avoue que je n'ai pas trouvé de réelle utilité sur ce point, mais c'est obligatoire, on va donc crééer un environement vide.
{.is-info}


L'environnement vide mets en place de la façon suivante avec les brackets ```{}``` :

![ansible-semaphore-config-02.png](/ansible-semaphore/configuring-semaphore/ansible-semaphore-config-02.png)

## Configuration de l'inventaire

Ici, cela se passe comme pour Ansible, configurez un inventaire (statique dans mon cas) avec vos différentes machines.

> L'inventaire de Semaphore est extrèmement limité ici, la plupart des options d'Ansible ne seront pas prises en compte comme les variables pour WinRM ou les ports spéciaux, on en reparle dans les bonus.
{.is-warning}

![ansible-semaphore-config-03.png](/ansible-semaphore/configuring-semaphore/ansible-semaphore-config-03.png)


## Configuration du dépôt pour les playbook

### Configuration d'une clé vide

Dans notre cas, nous allons utiliser un dépôt pointant sur un dossier local, pour que cela soit reconnu par Semaphore, créez d'abord une clé de type **none** dans le **Key Store** comme ceci : 

![ansible-semaphore-config-04.png](/ansible-semaphore/configuring-semaphore/ansible-semaphore-config-04.png)

### Configuration du dépôt

Maintenant, configurez votre dépôt.
Dans mon cas, j'utilise mon dépôt existant du serveur Ansible comme ceci :

![ansible-semaphore-config-05.png](/ansible-semaphore/configuring-semaphore/ansible-semaphore-config-05.png)


# Utilisation de Semaphore

Pour utiliser ce qu'on appelle un "playbook" dans Ansible, on passe ici par les **Task Templates**.
Cela intègre en fait la programmation complète du cron et du lien entre les différents playbook et inventaires que vous aurez créer.

Voici un exemple de template que j'utilise : 

![ansible-semaphore-config-06.png](/ansible-semaphore/configuring-semaphore/ansible-semaphore-config-06.png)


L'inventaire est le suivant (une partie seulement) : 

```yaml
[vms]
localhost       ansible_connection=local
192.168.1.7
192.168.1.12
192.168.1.17

### EXT ###
#VPNAccess
vpnaccess

#Wazuh
wazuh

#Webhost
webhost
```

Le playbook est le suivant : 
```yaml
- hosts: vms
  become: yes
  become_method: sudo
  tasks:
   - name: updates a server
     apt: update_cache=yes

   - name: upgrade a server
     apt: upgrade=dist dpkg_options='force-confold,force-confdef'

   - name: Check if a reboot is required
     register: reboot
     stat: path=/var/run/reboot-required get_md5=no

   - name: Reboot the services
     shell: needrestart -ra -l
     when: reboot.stat.exists == false

   - name: Reboot the server using reboot
     command: /sbin/reboot
     async: 1
     poll: 0
     when: reboot.stat.exists == true

   - name: Wait for systems to become reachable
     wait_for_connection:
```



# Bonus

## Utilisation de ports SSH autres que 22

Semaphore ne reconnait pas les ports autres que 22 pour le SSH par défaut.
Pour contourner cela, vous devrez créer des alias SSH avec la commande suivante : 
```bash
nano /home/nsw-ansible/.ssh/config
```

```bash
Host webhost
  HostName XXX.XXX.XXX.XXX
  Port 2204

Host wazuh
  HostName XXX.XXX.XXX.XXX
  Port 2200

Host vpnaccess
  HostName XXX.XXX.XXX.XXX
  Port 2203
```

Utilisez ensuite les hôtes directement dans l'inventaire, comme dans mon exemple de la partie précédente.


## Connexion à des machines Windows via WinRM

Dans la partie **Key Store**, créez un accès de type **Login With password** dans un premier temps.
Vous l'aurez deviné, on parle de l'utilisateur et du mot de passe Windows : 

![ansible-semaphore-config-07.png](/ansible-semaphore/configuring-semaphore/ansible-semaphore-config-07.png)

Ensuite, dans vos inventaires Windows, ajoutez systématiquement les variables suivantes :
```yaml
[windows]
windows.ad.com

[windows:vars]
ansible_port=5986
ansible_connection=winrm
ansible_winrm_server_cert_validation=ignore
```

Cela donne ceci : 

![ansible-semaphore-config-08.png](/ansible-semaphore/configuring-semaphore/ansible-semaphore-config-08.png)


Ensuite, la programmation se fait de la même manière que des playbook pour du Linux.

Voici un exemple de playbook utilisé pour la mise à jour des serveurs Windows : 

```yaml
- hosts: windows
  tasks:
   - name: Check value for RebootPending
     win_command: Powershell.exe "Test-Path 'HKLM:\Software\Microsoft\Windows\CurrentVersion\Component Based Servicing\RebootPending'"
     register: rebootpending
   - debug:
       msg: "Value for RebootPending: {{rebootpending.stdout_lines}}"
   - name: Reboot if RebootPending value is True
     win_reboot:
       reboot_timeout_sec: 3600
     when: rebootpending.stdout.find("True") != -1

   - name: install all critical and security updates on virtual machine
     win_updates:
       category_names:
       - CriticalUpdates
       - SecurityUpdates
       state: installed
     register: update_result

   - name: reboot if required
     win_reboot:
       shutdown_timeout_sec: 3600
       reboot_timeout_sec: 3600
     when: update_result.reboot_required
```