---
title: Create an object store container in Infomaniak Public Cloud
description: Creating a s3 compatible container in CLI
published: true
date: 2022-02-18T13:29:33.063Z
tags: infomaniak, cloud, s3
editor: markdown
dateCreated: 2022-02-18T13:28:13.570Z
---

# Introduction
For this procedure, we use Debian 11.

> All the secrets and IDs in this procedure have been auto-generated, they don't exist.
{.is-info}


# Prerequisites
First of all, grab your openrc config file for the cli connection here: https://api.pub1.infomaniak.cloud/horizon/identity/application_credentials/

Then, install openstack client and AWS CLI:

```bash
apt install openstack-clients awscli
```


# Connecting to the CLI

Source the openrc connection file you download just before like this:

```bash
source /home/nicolas/app-cred-ns-openstack-openrc.sh
```
Then list your projects to be sure the connection is working:

```bash
openstack project list
```
You should have this kind of output:

```bash
+----------------------------------+-------------+
| ID                               | Name        |
+----------------------------------+-------------+
| 26d03k3155k4o4uhxe0lxnc33z07cm3a | PCP-89V66JZ |
+----------------------------------+-------------+
```

# Creating S3 credentials and connection to S3

Launch the following command:

```bash
openstack ec2 credentials create
```

It will give you this kind of output, save it.

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

Now, configure your AWS client (for speaking the S3 protocol):
```bash
aws configure
```

You will need your Access Key and Secret key from the step before:

```bash
root@abyssproject:~# aws configure
AWS Access Key ID [None]: p6b9xj7xlw2o12y1y34d9h3o58621ykk
AWS Secret Access Key [None]: 6691o7rc1u59owb7lu722603ujn4mjda
Default region name [None]:
Default output format [None]:
```

# Container creation
Launch the following command to create the bucket, replace "customer" in the end by the name you want:

```bash
aws --endpoint-url=https://s3.pub1.infomaniak.cloud s3api create-bucket --bucket customer
```

You will have this kind of output if it succeed:
```json
nicolas@abyssproject:~$ aws --endpoint-url=https://s3.pub1.infomaniak.cloud s3api create-bucket --bucket customer
{
    "Location": "/customer"
}
```

You can check that the container is well created with this command:

```bash
aws --endpoint-url=https://s3.pub1.infomaniak.cloud s3api list-buckets
```

You will have this output:
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

# Using S3 bucket

You only need those 3 things to use the newly created bucket:

- access: p6b9xj7xlw2o12y1y34d9h3o58621ykk                                                         
- secret: 6691o7rc1u59owb7lu722603ujn4mjda
- region us-east-1  (Default right now in Infomaniak Public Cloud)
