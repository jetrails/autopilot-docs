Magento can keep everything under `pub/media` in an S3 bucket instead of on your deployment's disk.
NGINX then serves those files straight out of the bucket, so most image requests never touch PHP or your shared storage.
This guide walks you through enabling it on an existing deployment.

!!! Assumptions:
Your site's primary domain name is `example.com` and you are logged in to the jump host as the cluster user.
We also assume your media bucket is named `jrc-123-medias3bucket-456`.
!!!

## Request The Bucket

The bucket isn't something you can create yourself, so open a ticket with JetRails support and ask for an S3 media bucket for your deployment.
We add the bucket, its access policy, and the instance permissions to your deployment's infrastructure.

Once that's done, the bucket name shows up on your deployment's Overview tab under **Media S3 Bucket**.
It looks something like this:

```
jrc-123-medias3bucket-456
```

You'll need that name for the rest of this guide.
Everything else happens over SSH on the jump host.

## Check How Much Media You Have

This is what decides how long the upload takes, so check it before you start anything else.

```shell
du -sh /var/www/example.com/shared/pub/media
find /var/www/example.com/shared/pub/media -type f | wc -l
```

A store with 100 GB of media takes several hours to upload, so plan around that.

## Point Magento At The Bucket

Change into the site's live directory.

```shell
cd /var/www/example.com/live
```

Turn off database media storage first.

```shell
php bin/magento config:set system/media_storage_configuration/media_database 0 --no-interaction
```

Then enable the S3 driver.

```shell
php bin/magento setup:config:set --no-interaction \
    --remote-storage-driver=aws-s3 \
    --remote-storage-bucket=jrc-123-medias3bucket-456 \
    --remote-storage-region=us-east-1
```

No access key or secret is needed here.
The server authenticates to S3 using its instance role.

Confirm it got written.

```shell
grep -A6 remote_storage app/etc/env.php
```

The driver value should read `aws-s3` now instead of `file`.

## Upload Existing Media

This copies everything in `pub/media` up to the bucket.
It runs for a long time on a large store, so start it detached and it will survive your SSH session closing.

```shell
cd /var/www/example.com/live
setsid nohup php bin/magento remote-storage:sync --no-interaction \
    > ~/remote-storage-sync.log 2>&1 < /dev/null &
```

You can watch it with either of these.

```shell
tail -f ~/remote-storage-sync.log
aws s3 ls s3://jrc-123-medias3bucket-456 --recursive --summarize | tail -3
```

The command is safe to re-run.
If it gets interrupted, just start it again.

Files land under a `media/` prefix in the bucket, mirroring your `pub/media` folder.

```
media/catalog/product/a/b/example.jpg
```

Let this finish before you move on, so NGINX isn't serving out of a half-filled bucket.

## Serve Media Through NGINX

Your NGINX config is generated from a template by the `vhost` command, so anything you edit directly in the config files gets overwritten the next time it regenerates.
Instead you set a parameter on the site and let `vhost` rewrite the config for you.

To see which template your site uses and which parameters it takes:

```shell
vhost list
vhost template info jrc-magento2
```

The parameter you want is `s3_media_bucket_name`, described as "S3 bucket name for media storage, blank to disable".
Support for it is built into the standard Magento template, so nothing has to be added to your deployment.
Setting it just switches which version of the media block gets generated.

!!! Older Deployments
If `vhost template info` doesn't list `s3_media_bucket_name` at all, your deployment is running an older machine image from before this support was added.
Mention that on your ticket when you request the bucket and we'll update your deployment's templates for you.
!!!

To see what your site is set to right now:

```shell
vhost info example.com
```

Before modifying, run `vhost diff example.com` to see if your NGINX configs deviated from the template's original output.
Once you are comfortable moving forward, set the bucket name:

```shell
vhost modify example.com s3_media_bucket_name=jrc-123-medias3bucket-456
```

Here's what that actually changes.
Inside `location /media/`, the block matching image and asset extensions currently serves files off local disk:

```nginx
location ~* \.(ico|jpg|jpeg|png|gif|svg|svgz|webp|avif|avifs|js|css|eot|ttf|otf|woff|woff2)$ {
    add_header Cache-Control "public";
    expires +1y;
    try_files $uri $uri/ /get.php$is_args$args;
}
```

After the change, that block proxies the request to your bucket instead:

```nginx
location ~* \.(ico|jpg|jpeg|png|gif|svg|svgz|webp|avif|avifs|js|css|eot|ttf|otf|woff|woff2)$ {
    # Proxying to AWS S3 storage.
    resolver 8.8.8.8;
    set $bucket "jrc-123-medias3bucket-456";
    proxy_pass https://s3.amazonaws.com/$bucket$uri;
    proxy_pass_request_body off;
    proxy_pass_request_headers off;
    proxy_intercept_errors on;
}
```

There's no `try_files` fallback in the new block, which is why the upload has to finish first.

Confirm the regenerated config contains it.

```shell
grep -A5 "Proxying to AWS S3" /etc/nginx/conf.d/example.com/magento.conf
```

If that comes back empty then the parameter didn't apply, so re-check `vhost info example.com` before going any further.

`/etc/nginx` points at shared storage so every web node sees the regenerated config right away, but each one still needs a reload before it takes effect.

```shell
cluster exec --role web -- sudo nginx -t
cluster exec --role web -- sudo systemctl reload nginx
```

Reload php-fpm too so it picks up the new configuration, and flush Magento's cache.

```shell
cluster exec --role web -- sudo systemctl reload php-fpm
cd /var/www/example.com/live && php bin/magento cache:flush
```

## Test It

Pick any file you can see in the bucket and note its size.

```shell
aws s3 ls s3://jrc-123-medias3bucket-456/media/catalog/product/a/b/
```

Then request that same file through the site and compare.

```shell
curl -s -o /dev/null -w '%{http_code} %{content_type} %{size_download}\n' \
    https://example.com/media/catalog/product/a/b/example.jpg
```

A 200, the right content type, and a size that matches the bucket means media is being served from S3.

Now check that writing works.
Upload an image in the Magento admin, either on a product or under Content > Media Gallery, and confirm it shows up in the bucket.

```shell
aws s3 ls s3://jrc-123-medias3bucket-456/media/ --recursive | tail
```

## Things To Know

Keep your local `pub/media` folder in place until you're confident the site is running correctly on S3.
It's your fallback, and it's what lets you revert at any point.

## Reverting

To go back to local media storage:

```shell
cd /var/www/example.com/live
php bin/magento setup:config:set --no-interaction --remote-storage-driver=file
php bin/magento cache:flush
vhost modify example.com s3_media_bucket_name=
cluster exec --role web -- sudo nginx -t
cluster exec --role web -- sudo systemctl reload nginx
```
