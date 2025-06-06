---
title: Serve Web Files on Root Filesystem
---

Web files are accessible across your cluster deployment using NFS.
While this provides a convenient way to share files, it can also introduce challenges, particularly around old code bases that still use modules that follow the PSR-0 standard.
On newer deployments of Magento 2, this shouldn't be an issue, but if you are working with older code bases, you may need to serve web files from the root filesystem instead of NFS.

On newer deployments of Magento 2 (template version 5.2 and later), you can convert your cluster to serve web files from the root filesystem.
This will resolve any slowness that you might feel when accessing the admin interface but it does come with some caveats.

## Conversion walkthrough

!!! Assumptions:
You are logged in as the cluster user (uid 9001) and your site's primary domain name is **example.com**.
This feature is also only compatible with Magento at the moment, since other applications are not structured in a way that would support this type of setup.
!!!

First, we will need to modify the nginx configuration for your site to change where the root directory is set.
Luckily, this is a simple process that can be done with the `vhost` command.

```
vhost modify default-backend enable_sync_healthcheck=yes
vhost modify example.com use_www_cache_root=yes
```

If you made a lot of customizations to your nginx configs and deviated from the base template, then you may want to reimplement the changes modifing the nginx configs or implement the root change manually. You can review how much your configs deviate from the base template by running the following command:

```
vhost diff example.com
```

Next, we will need to create a custom boot hook that will sync the web files from the NFS share to the root filesystem on boot.

```bash
sudo cp -p /opt/jrc/hooks-available/boot.d/93-sync-www-data-on-boot /mnt/jrc-comms/hooks/boot.d/
```

Now all that is left is to execute the custom boot hook script to sync the web files from the NFS share to the root filesystem.
The script also handles reloading nginx and restarting php-fpm.

```bash
cluster exec --role web -- sudo /mnt/jrc-comms/hooks/boot.d/93-sync-www-data-on-boot
```

## Boot process

Everything is handled in the boot hook script.
The script syncs over web files for every site defined in `vhost` that has the `use_www_cache_root` option set to `yes`.
Here are some high level steps of what happens during the boot process:

1. Remove existing cache in `/var/www-cache` (faster copy process)
2. Ensure that the web node can still serve web files by serving files from the NFS share.
3. Start the copy process from the NFS share to the root filesystem.
4. Start serving web files from the root filesystem.

## Manually syncing changes

If you need to sync changes that are made in your NFS share to the root filesystem, you can run the following command:

```bash
cluster exec --role web -- sudo /mnt/jrc-comms/hooks/boot.d/93-sync-www-data-on-boot
```

This will run the same script that is executed during the boot process and will sync the web files from the NFS share to the root filesystem while still serving files from the NFS share until the copy process is complete.

If you would like to simply copy the files over without having the NFS fallback, you can run the following command:

```bash
cluster exec --role web -- sudo /opt/jrc/sbin/sync-www-cache example.com
```

## Manually rotate web nodes

If you find yourself in a situation where you need to manually rotate your web nodes one by one, then you can run the following command:

```bash
cluster exec --role web -- sudo bash -c 'www-health mark unhealthy && sleep infinity'
```

This command will run on every web node and mark the node as unhealthy, which will cause the target group to stop sending traffic to the node.
It will then fail health checks and a new web node will be started to replace it.
We wait till infinity because we want to wait for that node to be fully shutdown before continuing to replace the next web node.
