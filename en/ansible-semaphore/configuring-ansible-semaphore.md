---
title: Configuring Ansible Semaphore
description: Configuring Ansible Semaphore and starting the first tasks
published: true
date: 2024-01-04T10:05:17.290Z
tags: ansible, semaphore
editor: markdown
dateCreated: 2024-01-04T10:05:17.290Z
---

# Introduction

The goal here is to configure Ansible Semaphore and start using it, Ansible Semaphore is a GUI for Ansible: https://www.semui.co/

> The installation is documented here: https://wiki.abyssproject.net/fr/ansible-semaphore/installing-semaphore-debian
{.is-info}



# Configuring Semaphore

## Key store setup

The first step is to configure a Key Store for Semaphore.
Here, I use the same SSH key to connect to all my Linux servers, the user is root.

Go to **Key Store** -> **New Key** and enter the private key in text format.
Specify the user like this: 
 
![ansible-semaphore-config-01.png](/ansible-semaphore/configuring-semaphore/ansible-semaphore-config-01.png)


## Configuring the environment

The next step is the environment configuration.
> As the documentation of Semaphore is empty about this and what you can do with it, just create an empty environment.
{.is-info}


You can set up it with empty brackets ```{}```:

![ansible-semaphore-config-02.png](/ansible-semaphore/configuring-semaphore/ansible-semaphore-config-02.png)

## Inventory


For this step, it's like configuring the Ansible inventory (I use a static one).

> The inventory is extremely limited, most of the Ansible option are not taken in account here, like WinRM or special port, we talk about this in the bonus.
{.is-warning}

![ansible-semaphore-config-03.png](/ansible-semaphore/configuring-semaphore/ansible-semaphore-config-03.png)


## Repositories configuration for the playbooks

### Empty key creation

We are going to configure a local repository, for this, we need to create a **none** key type in the **Key Store** like this: 

![ansible-semaphore-config-04.png](/ansible-semaphore/configuring-semaphore/ansible-semaphore-config-04.png)

### Repository creation

Now, create your repository, I use the existing Ansible one for my case:

![ansible-semaphore-config-05.png](/ansible-semaphore/configuring-semaphore/ansible-semaphore-config-05.png)


# Utilisation de Semaphore

To use an Ansible Playbook, we need to set up **Task Templates** in Semaphore.
It manages completely cron programmation, links between playbooks and inventory and environment management.

Here is a template that I use: 

![ansible-semaphore-config-06.png](/ansible-semaphore/configuring-semaphore/ansible-semaphore-config-06.png)


Here is an extract of my inventory: 

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

The playbook is the following: 
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

## Using custom ssh ports

Semaphore doesn't recognize custom ports for SSH.
To bypass this, you can create a custom SSH alias config: 
```bash
nano /home/ansible/.ssh/config
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

Then, use the host name directly in the inventory like my previous example.

## Windows machine connection via WinRm

In the **Key Store**, create an access type **Login With password**.
As you will have found, you need to put the Windows login and password here: 

![ansible-semaphore-config-07.png](/ansible-semaphore/configuring-semaphore/ansible-semaphore-config-07.png)

Then, in your inventory, add the following vars for Windows hosts:
```yaml
[windows]
windows.ad.com

[windows:vars]
ansible_port=5986
ansible_connection=winrm
ansible_winrm_server_cert_validation=ignore
```

You will have something like this: 

![ansible-semaphore-config-08.png](/ansible-semaphore/configuring-semaphore/ansible-semaphore-config-08.png)


Task templates are then created in the same way as we do for linux.

Here is a playbook that I use for updating Windows servers: 

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