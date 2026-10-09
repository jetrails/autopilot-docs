Once your media lives in an S3 bucket, you can't just `cd` into `pub/media` to look around anymore.
The `media-bucket` command on your jump host mounts the bucket as a folder for a little while, so you can browse it and copy files in and out with the tools you already use.
It's meant for quick jobs like checking that an image made it up, or pulling down a handful of files.
It is not a permanent replacement for `pub/media`.

!!! Assumptions:
Your site's primary domain name is `example.com` and you are logged in to the jump host as the cluster user.
We also assume your media bucket is named `jrc-123-medias3bucket-456`.
For the SFTP and rsync examples, your cluster user is `jrc-abcd-1234` and your deployment's elastic IP address is `1.2.3.4`.
!!!

## Requirements

Your deployment needs a media S3 bucket.
If you don't have one yet, follow the [Enable S3 Media Storage For Magento](/guides/enable-s3-media-storage-for-magento/) guide first.

The command only exists on the jump host.
It mounts the bucket there and nowhere else.

!!! Older Deployments
If `/opt/jrc/sbin/media-bucket` doesn't exist on your jump host, your deployment is running an older machine image from before this command was added.
Open a ticket with JetRails support and we'll get your deployment updated.
!!!

## Mount The Bucket

```shell
sudo /opt/jrc/sbin/media-bucket mount
```

You'll see a warning every time you mount it.

```
WARNING: Do NOT leave media bucket mounted. Unmount when you are done with it by running:
         "/opt/jrc/sbin/media-bucket unmount"

bucket jrc-123-medias3bucket-456 is mounted at /mnt/jrc-123-medias3bucket-456
bucket is available at /var/www/example.com/media-bucket
```

The bucket gets mounted at `/mnt/jrc-123-medias3bucket-456` and a link to it is placed in your site's folder.
Magento keeps everything under a `media/` prefix, so your files show up here:

```shell
ls /var/www/example.com/media-bucket/media/catalog/product
```

Everything in the mount is owned by the cluster user and the `jetrails` group.
That means you can work with it as the cluster user without needing `sudo`.

Running `mount` again while the bucket is already mounted is safe.
It just tells you it's already mounted and makes sure the link is in place.

## Unmount The Bucket

When you're done, unmount it.

```shell
sudo /opt/jrc/sbin/media-bucket unmount
```

This unmounts the bucket, removes the link from your site's folder, and cleans up the empty mount folder.
It's safe to run even if the bucket isn't mounted anymore, which is also how you clean up the leftover link after a reboot.

The bucket disappears from `/mnt` and your site's folder right away, even if something is still using it.
Anything that already had a file or folder open, like a copy that's still running or a shell sitting in one of the bucket's folders, keeps working until it's done.
The mount goes away for good once nothing is using it anymore.

## Why It's Temporary

The mount isn't a real disk.
Every directory listing, every file you look at, and every file you open turns into a request to S3.
That makes things like `du` or `find` across your whole media folder slow on a large store.

It also costs money.
AWS bills S3 by the request, so a `du` or a `find` across the whole bucket can add up to thousands of billed requests in one go.
Stick to the folders you actually need.

That goes for the Code Editor too.
Its search follows the link by default, so searching your whole workspace while the bucket is mounted reads through the entire bucket.
Either unmount first, or add `media-bucket` to the search's "files to exclude" box.

The mount only lives on the jump host.
Your web nodes share `/var/www` with the jump host, so they see the `media-bucket` link too, but it's broken on their end.
Nothing on your site should ever be pointed at it.

And it doesn't come back after a reboot or after your jump host gets replaced.
The link stays behind in your site's folder, pointing at nothing, until you run `unmount`.

## What You Can Do With It

You can read files, copy new files in, replace existing files, and delete files.

```shell
cp ~/new-banner.jpg /var/www/example.com/media-bucket/media/wysiwyg/
cp /var/www/example.com/media-bucket/media/catalog/product/a/b/example.jpg ~/
rm /var/www/example.com/media-bucket/media/wysiwyg/old-banner.jpg
```

The link goes in your site's folder on purpose.
That's the folder the Code Editor (VS Code in your browser) opens, so while the bucket is mounted you'll see `media-bucket` in its file explorer.
You can browse the bucket from there, open files, drag new files in, and delete them, without touching a terminal.

You can also reach it from your own computer over rsync or SFTP.
Both have a couple of gotchas, so they get their own [examples](#examples) below.

Some things you'd normally do with files don't work here.
That's because S3 doesn't store files, it stores objects, and the mount only does what S3 can actually do.

You can replace a file, but you can't edit part of one or add to the end of it.
S3 objects can't be changed once they're written, only replaced whole.
Copying a new version over a file works, and so does saving a file you opened in the Code Editor, because both write out the whole file again.

Renaming and moving files inside the bucket don't work either.
S3 has no rename.
Renaming a file would really mean copying it to a new name and deleting the old one, and renaming a folder would mean doing that for every file inside it.
The mount won't fake that for you, so copy the file to its new path and delete the old one yourself.

You also can't change permissions or ownership on anything in the mount, and you can't create symlinks inside it.
S3 objects don't have an owner, permissions, or a way to be a symlink.
That's why every file shows up with the same owner and permissions no matter what.

Folders aren't real in S3 either.
A folder only exists because there are files in it, and it goes away on its own once the last one is deleted.
So `rm -r` on a folder may complain that it can't remove some of the folders inside it.
The files are still deleted, and the folders are already gone.

## Examples

These run from your own computer, not the jump host.
Mount the bucket on the jump host first, and unmount it when you're done.

rsync needs a couple of extra flags to upload into the bucket.

```shell
rsync -rv --inplace --ignore-existing ./banners/ \
    jrc-abcd-1234@1.2.3.4:/var/www/example.com/media-bucket/media/wysiwyg/banners/
```

`--inplace` makes rsync write straight to the final file.
Without it, rsync writes to a temporary file and renames it into place, and that rename fails.

`--ignore-existing` makes rsync skip anything that's already in the bucket.
rsync can't update a file that's already there, so without this flag it tries to and fails on each one.
Running the same command again only uploads files the bucket doesn't have yet.

Use `-r` instead of `-a` when uploading.
`-a` also tries to set modification times on the files, which the bucket refuses.
The files still upload, but rsync reports `failed to set times` for each one and exits with an error.

If you need to replace a file that's already in the bucket, upload it with SFTP instead, or copy it over on the jump host with `cp`.

Downloading works like any other rsync.

```shell
rsync -av jrc-abcd-1234@1.2.3.4:/var/www/example.com/media-bucket/media/wysiwyg/banners/ ./banners/
```

With SFTP, connect as the cluster user and change into the bucket.

```shell
sftp jrc-abcd-1234@1.2.3.4
```

```
sftp> cd /var/www/example.com/media-bucket/media/wysiwyg
```

Upload a file, download one, and delete one.

```
sftp> put new-banner.jpg
sftp> get old-banner.jpg
sftp> rm old-banner.jpg
```

Running `put` again with a newer version of the same file replaces the one in the bucket.

To upload a whole folder, use `put -r`.

```
sftp> put -r banners
Entering banners/
Entering banners/sale
remote setstat "/var/www/example.com/media-bucket/media/wysiwyg/banners/sale": Permission denied
remote setstat "/var/www/example.com/media-bucket/media/wysiwyg/banners": Permission denied
```

The `Permission denied` lines are SFTP trying to set permissions on each folder after it uploads them, which the bucket doesn't support.
You can ignore them.
The files all made it up.

You can also use a desktop SFTP client like FileZilla or WinSCP.
Connect with the same user and IP address and open `/var/www/example.com/media-bucket`.
If your client uploads to a temporary file name and renames it once it's done, turn that off, since renames don't work here.
WinSCP does this by default for larger files.
You can turn it off under Preferences > Transfer > Endurance by setting "Transfer to temporary filename" to "Disable".
