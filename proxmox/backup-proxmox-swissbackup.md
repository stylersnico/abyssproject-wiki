---
title: Sauvegarde des machines Proxmox sur Swissbackup
description: Sauvegarde des machines Proxmox sur Swissbackup via S3FS
published: true
date: 2022-10-27T08:14:19.883Z
tags: infomaniak, s3, s3fs, proxmox
editor: markdown
dateCreated: 2022-10-27T08:14:19.883Z
---

# Introduction

Le but de cet article est de mettre en place la sauvegarde des machines Proxmox sur le produit Swissbackup de Infomaniak via le protocole S3 et S3FS.

> Vous devez déjà avoir fait un espace de stockage S3 sur le produit Swissbackup : https://www.infomaniak.com/fr/swiss-backup
> Vous devez également avoir obtenu le **endpoint**, l'**access key** et la **secret key**.
{.is-info}


# Installation des pré-requis

 
Installez les pré-requis nécessaires directement sur votre hôte Proxmox :

```bash
apt install openstack-clients awscli s3fs
```

# Configuration du bucket

Lancez la commande suivante pour configurer votre client S3: 
```bash
aws configure
```

Rentrez uniquement votre **access key** et la **secret key** :
```bash
root@pve01:/mnt/s3-swissbackup# aws configure
AWS Access Key ID [None]: XXXXXXXXXXXXX
AWS Secret Access Key [None]: XXXXXXXXXXXXX
Default region name [None]:
Default output format [None]:
root@pve01:/mnt/s3-swissbackup#
```


Maintenant, créez un bucket avec la commande suivante (adaptez l'url du **endpoint** si nécessaire) :
```bash
aws --endpoint-url=https://s3.swiss-backup03.infomaniak.com s3api create-bucket --bucket backup
```

Vous devriez avoir le retour suivant : 
```bash
root@pve01:/mnt/s3-swissbackup# aws --endpoint-url=https://s3.swiss-backup03.infomaniak.com s3api create-bucket --bucket backup
{
    "Location": "/backup"
}
root@pve01:/mnt/s3-swissbackup#
```

# Montage du système S3

Créez d'abord les dossiers nécessaire : 

```bash
mkdir /mnt/s3-swissbackup
mkdir /etc/s3fs
```

Créez un fichier qui inclue vos informations de connexion au bucket S3 : 
```bash
echo accesskey:secretkey > /etc/s3fs/.passwd-s3fs
chmod 600 /etc/s3fs/.passwd-s3fs
```


Montez enfin le système de fichier avec la commande suivante (adaptez l'url du **endpoint** si nécessaire) :
```bash
s3fs backup /mnt/s3-swissbackup -o passwd_file=/etc/s3fs/.passwd-s3fs  -o url=https://s3.swiss-backup03.infomaniak.com -o use_path_request_style  -o umask=0002
```

Vous pouvez mettre cette commande dans un script pour le montage au démarrage :

```bash
echo "sleep 10" > /root/mount-s3fs.sh
echo "s3fs backup /mnt/s3-swissbackup -o passwd_file=/etc/s3fs/.passwd-s3fs  -o url=https://s3.swiss-backup03.infomaniak.com -o use_path_request_style  -o umask=0002" >> /root/mount-s3fs.sh
chmod +x /root/mount-s3fs.sh
```

Ouvrez votre crontab :
```bash
crontab -e
```
Ajoutez la ligne suivante : 
```bash
@reboot /bin/sh /root/mount-s3fs.sh
```

# Configuration dans Proxmox 

Allez dans l'interface Proxmox, dans **Datacenter** -> **Storage** -> **Add**  -> **Directory** : 

![proxmox-swissbackup-01.png](/proxmox/swissbackup/proxmox-swissbackup-01.png)

Configurez le dossier comme ce qu'il suit, pensez-bien à cocher **Shared** :

![proxmox-swissbackup-02.png](/proxmox/swissbackup/proxmox-swissbackup-02.png)


Vous pouvez également configurer une rétention directement dans l'interface.

Il ne vous restera plus qu'à mettre en place le job de sauvegarde en utilisant ce repository :)

## Sources

- https://docs.infomaniak.cloud/documentation/04.object-storage/third-party-integration/02.s3fs/