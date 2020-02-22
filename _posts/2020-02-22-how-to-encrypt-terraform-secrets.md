---
layout: post
title: "Encrypting secrets in Terraform"
date: 2020-02-22 17:10:01
image: '/assets/img/'
description: How to encrypt Terraform secrets using gpg
tags:
- Terraform 
- Terraform secrets
- Encrypt
- GPG
- AWS
categories:
- Terraform
twitter_text:
---

## Main issue

Terraform is a great tool for infra structure as code. I use it quite a lot for creating resources in AWS e.g. EC2, RDS instances. However, there are some resources in AWS that require passwords before Terraform can create it e.g. RDS instance or ElastiCache instance.

Of course you can define the user and the password as Terraform variables but these credentials will be plain text inside your project.

For instance if  we want to create an RDS instance this what it would look like in Terraform:


```terraform
resource "aws_db_instance" "default" {
  allocated_storage    = 20
  storage_type         = "gp2"
  engine               = "mysql"
  engine_version       = "5.7"
  instance_class       = "db.t2.micro"
  name                 = "mydb"
  username             = "foo"
  password             = "foobarbaz"
  parameter_group_name = "default.mysql5.7"
}
```


As you can see the password `foobarbaz` will be stored as plain text in your terraform project.



## Solution

### Step 1. Create secrets directory


Create a `secrets` directory which will contains all sort of sensitive data used in Terraform. In this example we will focus on encrypting one secret i.e. RDS instance password.

```bash
mkdir secrets

echo "{ \"password\": \"foobarbaz\" }" >> secrets/rds.json
```

<br/>

### Step 2. Get secrets from the json file

```terraform
data "external" "rds" {
  program  = [ "cat", ".secrets/rds.json"]
}


resource "aws_db_instance" "default" {
  allocated_storage    = 20
  storage_type         = "gp2"
  engine               = "mysql"
  engine_version       = "5.7"
  instance_class       = "db.t2.micro"
  name                 = "mydb"
  username             = "foo"
  password             = "${data.external.rds.result.password}"
  parameter_group_name = "default.mysql5.7"
}
```

<br/>

### Step 3. Encryption

To encrypt the secret file i.e. **rds.json** we can use **gpg** and encrypting the file either with a key or passphrase.

```bash

gpg -o secrets/rds.json.gpg \
      --batch \
      --symmetric \
      --openpgp \
      --cipher-algo AES256 \
      --s2k-cipher-algo AES256 \
      --s2k-digest-algo SHA512 \
      --s2k-mode 3 \
      --s2k-count 65011712 \
      --armor \
      --emit-version \
      secrets/rds.json

```
<br/>

Then you will be prompted for entering a pass-phrase twice.

This is what the output file `rds.json.gpg` will look like after encryption

```bash

cat secrets/rds.json.gpg
-----BEGIN PGP MESSAGE-----
Version: GnuPG/MacGPG2 v2

jA0ECQMKrGbmVhBpU7L/0lkBkpKcI8nXpiWDGKV7qdBDOt2Gb4qGvCu9aUkYdhcH
cwRsCBFyqDQRIfQSTmc1NFfr/3HJyADQKOBoftnCC1TGYYOL7AhexGhl0NaemqH8
zl1GOz3b1QHETg==
=Gyon
-----END PGP MESSAGE-----
```

<br/>

### Step 4. Wrap up

As you noticed the `secrets` directory will contains two files one is encrypted `rds.json.gpg` and one is not encrypted `rds.son`

Now we need to add the un-encrypted file to `gitignore` so it will not be committed.

**.gitignore:**
```bash
**/secrets/*
!**/secrets/*.gpg
```

<br/>

### Extra

To make this more efficient, you can create an init function that decrypts all secrets in your project. This way all secrets will be available for Terraform and you won't get errors when running Terraform commands e.g. `terraform plan`, `terraform apply`