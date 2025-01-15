---
title: Onboard Magento With Deployer
---

This guide will walk you through onboarding your existing Magento Store onto the AutoPilot platform using Deployer.
We will initially deploy a sample codebase onto a new deployment.
Although we demonstrate using PHP Deployer, you can use any deployment tool you prefer.
Filesystem requirements are outlined in the [Filesystem Requirements](#filesystem-requirements) section.

!!! Assumption:
We assume the store's domain name is `example.com`.
!!!

## Create Deployment

Create a new deployment through the AutoPilot dashboard that matches your Magento version to ensure compatibility with services like PHP and MySQL.
If your Magento version is unavailable, mix and match service versions to meet your requirements.
If provisioning fails, open a support ticket for assistance.

After provisioning, visit the Security tab to whitelist your IPv4 address and SSH key.
Find your public IP at [ip.jetrails.com](https://ip.jetrails.com).

[!ref target="blank" text="My IP Address"](https://ip.jetrails.com)

Your shell access command is available on the Overview tab.

!!! Assumption:
We assume the shell access command is `ssh jrc-3p7i-376i@10.10.10.10`.
!!!

## Deployer File

We created an AutoPilot recipe for Deployer to aid the deployment process.
Find it on GitHub at [jetrails/deployer-autopilot](https://github.com/jetrails/deployer-autopilot).
A sample deployer recipe is in the `examples` folder, e.g., [magento2.php](https://github.com/jetrails/deployer-autopilot/blob/master/examples/magento2.php).

Modify the following settings in the deployer file with your values:

!!! Assumption:
We assume the repository is `git@github.com:example/example.git` and you use deploy keys for GitHub authentication.
!!!

```php
set("repository", "git@github.com:example/example.git");
set("primary_domain", "example.com");
set("cluster_user", "jrc-3p7i-376i");
set("elastic_ip", "10.10.10.10");
```

Find your cluster user's public SSH key by connecting to your deployment via SSH and running:

```shell
cat ~/.ssh/id_rsa.pub
```

## Import Database

Upload your database dump to your deployment using `rsync`:

!!! Assumption:
Your database dump is named `database-dump.sql`.
!!!

```shell
rsync -aP database-dump.sql jrc-3p7i-376i@10.10.10.10:/var/www/example.com/
```

Import the database dump into your MySQL database.
Find your database name in the AutoPilot dashboard's Overview tab.

!!! Assumption:
We assume the database name is `vo889841yc249h86`.
!!!

Run the following command:

```shell
mysql -D vo889841yc249h86 < /var/www/example.com/database-dump.sql
```

## Upload Media Files

Upload your media files using `rsync`:

!!! Assumption:
Your local media folder is located at `/path/to/media/`.
!!!

```shell
rsync -aP --no-p --no-g --chmod=ugo=rwX /path/to/media/ jrc-3p7i-376i@10.10.10.10:/var/www/example.com/pub/media/
```

The `--no-p`, `--no-g`, and `--chmod=ugo=rwX` flags ensure proper permissions on the media files.

## Deploy Codebase

Once you have your deployer file, database, and media files ready, deploy your codebase:

```shell
dep deploy
```

If you integrated our example recipe, services like php-fpm and varnish will restart automatically after a successful deployment.

## Filesystem Requirements

NGINX and PHP-FPM run under the `www-data` user and group, therefore, all publicly accessible files must be readable by the `www-data` user.
Additionally, any directories or files that need to be writable must also have write permissions for the `www-data` user.
This is easy to do with Deployer's `acl` strategy, which leverages `setfacl` to set extended permissions.

To ensure proper ownership and permissions for the codebase files, follow these guidelines:

1. The codebase files should be owned by `jrc-3p7i-376i:jetrails`.
2. The codebase files should have the permissions `u=rwX,g=u,o=rX`.
3. The `www-data` user should have `rwX` permissions on writable directories such as `var`.

If you find yourself needing to set ownership and permissions manually, then follow the outlined steps.
Use the following commands to set the ownership and permissions:

```shell
sudo chown -R jrc-3p7i-376i:jetrails /var/www/example.com
sudo chmod -R u=rwX,g=u,o=rX /var/www/example.com
```

To grant the `www-data` user the necessary write permissions, use these commands:

```shell
sudo setfacl -L -R -m u:"www-data":rwX /var/www/example.com/live/{var,pub/static,pub/media,generated,var/page_cache}
sudo setfacl -dL -R -m u:"www-data":rwX /var/www/example.com/live/{var,pub/static,pub/media,generated,var/page_cache}
```