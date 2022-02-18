---
title: Créer un conteneur d'objet dans le public cloud Infomaniak
description: Création d'un conteneur compatible S3
published: true
date: 2022-02-18T13:54:07.223Z
tags: cloud, s3, info
editor: markdown
dateCreated: 2022-02-18T13:54:07.223Z
---

# Avant de commencer
Pour cette procédure, nous utilisons Debian 11.

> Tous les secrets ci-dessous sont générés de manière aléatoire, cela ne correspond pas à des identifiants existants.
{.is-info}


# Prérequis
Avant tout, récupérez la configuration openrc depuis votre espace Horizon : https://api.pub1.infomaniak.cloud/horizon/identity/application_credentials/

Après, installez le client openstack et le client AWScli :

```bash
apt install openstack-clients awscli
```


# Connexion à la ligne de commande

Chargez votre fichier de connexion openrc comme ceci :

```bash
source /home/nicolas/app-cred-ns-openstack-openrc.sh
```
Ensuite, utilisez cette commande pour lister vos projets Openstack afin d'être sûr que la connexion marche :

```bash
openstack project list
```
Vous devriez avoir une sortie comme ceci :

```bash
+----------------------------------+-------------+
| ID                               | Name        |
+----------------------------------+-------------+
| 26d03k3155k4o4uhxe0lxnc33z07cm3a | PCP-89V66JZ |
+----------------------------------+-------------+
```

# Création du compte S3 et connexion à S3

Lancez la commande suivante :

```bash
openstack ec2 credentials create
```

Cela vous donnera une sortie de ce genre, sauvegardez ces informations :

```bash
+------------+------------------------------------------------------------------------------------------------------------------------------------------------------+
| Field      | Value                                                                                                                                                |
+------------+------------------------------------------------------------------------------------------------------------------------------------------------------+
| access     | p6b9xj7xlw2o12y1y34d9h3o58621ykk                                                                                                                     |
| links      | {'self': 'https://api.pub1.infomaniak.cloud/identity/v3/users/ia40glrv2y88j3e91171w4fy0tz38fm7/credentials/OS-EC2/p6b9xj7xlw2o12y1y34d9h3o58621ykk'} |
| project_id | eo09d3oen9d8s72yex34y3j41i5j7o85                                                                                                                     |
| secret     | 6691o7rc1u59owb7lu722603ujn4mjda                                                                                                                     |
| trust_id   | None                                                                                                                                                 |
| user_id    | ia40glrv2y88j3e91171w4fy0tz38fm7                                                                                                                     |
+------------+------------------------------------------------------------------------------------------------------------------------------------------------------+
```

Maintenant, configurez votre client AWS (pour parler avec l'API S3) :
```bash
aws configure
```

Vous aurez besoin de votre Access Key et la Secret key de l'étape précédente :

```bash
root@abyssproject:~# aws configure
AWS Access Key ID [None]: p6b9xj7xlw2o12y1y34d9h3o58621ykk
AWS Secret Access Key [None]: 6691o7rc1u59owb7lu722603ujn4mjda
Default region name [None]:
Default output format [None]:
```

# Création du conteneur
Lancez la commande suivante pour créez le conteneur, remplacez le "customer" à la fin par le nom que vous voulez :

```bash
aws --endpoint-url=https://s3.pub1.infomaniak.cloud s3api create-bucket --bucket customer
```

Vous aurez ce genre de sortie si cela réussi :
```json
nicolas@abyssproject:~$ aws --endpoint-url=https://s3.pub1.infomaniak.cloud s3api create-bucket --bucket customer
{
    "Location": "/customer"
}
```

Vous pouvez lister vos conteneurs avec la commande suivante :

```bash
aws --endpoint-url=https://s3.pub1.infomaniak.cloud s3api list-buckets
```

Vous aurez ce genre de sortie :
```json
{
    "Buckets": [
        {
            "Name": "customer",
            "CreationDate": "2009-02-03T16:45:09.000Z"
        },
    ],
    "Owner": {
        "DisplayName": "PCP-89V66JZ:PCU-89V66JZ",
        "ID": "PCP-89V66JZ:PCU-89V66JZ"
    }
}
```

# Utilisation du conteneur S3

Vous aurez uniquement besoin de ces 3 informations dans votre application :

- access: p6b9xj7xlw2o12y1y34d9h3o58621ykk                                                         
- secret: 6691o7rc1u59owb7lu722603ujn4mjda
- region us-east-1  (Default right now in Infomaniak Public Cloud)
