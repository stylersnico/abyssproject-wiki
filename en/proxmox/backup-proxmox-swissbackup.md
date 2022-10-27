---
title: Backup Proxmox VMs on Swissbackup
description: Backup Proxmox VMs on Swissbackup via S3FS
published: true
date: 2022-10-27T08:23:47.322Z
tags: infomaniak, s3, s3fs, proxmox
editor: markdown
dateCreated: 2022-10-27T08:23:47.322Z
---

# Introduction

The goal of this guide is to set up the backup of proxmox virtual machines via the Swissbackup product of Infomaniak using S3 protocol and S3FS.

> You need to already have Swissbackup with a S3 storage: https://www.infomaniak.com/fr/swiss-backup
> You also need to have the **endpoint**, the **access key** and the **secret key**.
{.is-info}


# Prerequisites

 
Install the needed tools on your Proxmox host:

```bash
apt install openstack-clients awscli s3fs
```

# Configuring bucket

Launch the following command to configure your S3 client: 
```bash
aws configure
```

Enter only your **access key** and your **secret key**:
```bash
root@pve01:/mnt/s3-swissbackup# aws configure
AWS Access Key ID [None]: XXXXXXXXXXXXX
AWS Secret Access Key [None]: XXXXXXXXXXXXX
Default region name [None]:
Default output format [None]:
root@pve01:/mnt/s3-swissbackup#
```

Now, create the bucket with the following command (Adapt the **endpoint** url if needed):
```bash
aws --endpoint-url=https://s3.swiss-backup03.infomaniak.com s3api create-bucket --bucket backup
```

You should have the following return: 
```bash
root@pve01:/mnt/s3-swissbackup# aws --endpoint-url=https://s3.swiss-backup03.infomaniak.com s3api create-bucket --bucket backup
{
    "Location": "/backup"
}
root@pve01:/mnt/s3-swissbackup#
```

# Mounting the S3 file system

Create the needed directories: 

```bash
mkdir /mnt/s3-swissbackup
mkdir /etc/s3fs
```

Create a file that contains your S3 credentials: 
```bash
echo accesskey:secretkey > /etc/s3fs/.passwd-s3fs
chmod 600 /etc/s3fs/.passwd-s3fs
```


Now mount the file system with the following command (Adapt the **endpoint** url if needed):
```bash
s3fs backup /mnt/s3-swissbackup -o passwd_file=/etc/s3fs/.passwd-s3fs  -o url=https://s3.swiss-backup03.infomaniak.com -o use_path_request_style  -o umask=0002
```

You can put it in a script so the file system mount a launch:

```bash
echo "sleep 10" > /root/mount-s3fs.sh
echo "s3fs backup /mnt/s3-swissbackup -o passwd_file=/etc/s3fs/.passwd-s3fs  -o url=https://s3.swiss-backup03.infomaniak.com -o use_path_request_style  -o umask=0002" >> /root/mount-s3fs.sh
chmod +x /root/mount-s3fs.sh
```

Open your crontab:
```bash
crontab -e
```
Add the following line: 
```bash
@reboot /bin/sh /root/mount-s3fs.sh
```

# Proxmox configuration

Go to the Proxmox web interface in **Datacenter** -> **Storage** -> **Add**  -> **Directory**: 

![proxmox-swissbackup-01.png](/proxmox/swissbackup/proxmox-swissbackup-01.png)

Configure the folder like this, thick **Shared**:

![proxmox-swissbackup-02.png](/proxmox/swissbackup/proxmox-swissbackup-02.png)


You can also configure the backup retention directly in the interface.

Now you can set up the backup with this new repository :)

## Sources

- https://docs.infomaniak.cloud/documentation/04.object-storage/third-party-integration/02.s3fs/