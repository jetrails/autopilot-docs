---
title: Create A Custom Boot Hook
---

If you ever need to run custom tasks during the boot process of your instances, AutoPilot provides a flexible system for adding custom boot hook scripts.
These scripts allow you to execute specific commands or tasks at boot time, ensuring that your environment is configured exactly as needed.

In this article, we will walk through the process of creating such scripts and explain how they integrate with the existing boot sequence.
Along the way, we will cover best practices, troubleshooting tips, and an example of how to implement a custom boot hook script.

!!! Assumption:
You have root access and you have a shell open on your jump instance as the root user.
!!!

## Underlying Mechanism

Our platform executes a series of scripts on boot.
These scripts mostly reside in `/opt/jrc/hooks/boot.d/` and are executed in numerical order based on their filenames.
Before executing these scripts, the system checks for additional scripts in `/mnt/jrc-comms/hooks/boot.d/` and injects them into the execution order.
If you create a script with the same filename as an existing script in `/opt/jrc/hooks/boot.d/`, it will override the original script.
This allows you to extend or modify the boot process without altering the original scripts, ensuring your custom logic runs seamlessly alongside the default behavior.

The application runs scripts in ascending order based on their numeric prefixes. Scripts added to `/mnt/jrc-comms/hooks/boot.d` will be injected into `/opt/jrc/hooks/boot.d` based on script prefix automatically. For example:

- `05-mount-recovery-mount` runs first (found in `/opt/jrc/hooks/boot.d`)
- `10-cluster-clean` runs next (found in `/opt/jrc/hooks/boot.d`)
- `99-custom-task` runs last (added to `/mnt/jrc-comms/hooks/boot.d`)

**Key guidelines:**

- Prefixes determine execution order (e.g., `00-` to `99-`)
- Use two-digit numbering for clarity (e.g., `05-`, `10-`, `99-`)
- Scripts with the same prefix may execute in lexicographical order, but avoid ambiguity by using unique prefixes
- Scripts added to `/mnt/jrc-comms/hooks/boot.d` will persist AMI upgrades, stack clones, etc.

By following this structure, you can extend the boot process with custom logic while maintaining predictable execution order.
Adding scripts to persistent storage also prevents the need to reimage instances and replace them with new AMIs.

## Example: Hello World

In this example, we will simply create a custom boot hook script that will log a message to the screen.
First we will need to create a new file located at `/mnt/jrc-comms/hooks/boot.d/99-custom-task` with the following contents:

```bash
#!/bin/bash

set -e

# Import helper functions

. /opt/jrc/lib/helpers

# Preform task

debug "Hello World!"

```

Observe the following:

- The script has the prefix `99-`, which ensures it runs last in the boot sequence.
- It uses `set -e` to exit immediately if any command fails, preventing further execution in case of errors.
- It sources the helper functions from `/opt/jrc/lib/helpers` to utilize common utility functions like `debug`.

In order for the script to execute correctly, it must have the right permissions and ownership.
Each script must have the ownership of `root:root` and be executable. This is crucial for security and functionality and if not set correctly, then we refuse to run the script during the boot process.

You can run the following commands to set the correct permissions and ownership to our example script:

```shell
chown root:root /mnt/jrc-comms/hooks/boot.d/99-custom-task
chmod +x /mnt/jrc-comms/hooks/boot.d/99-custom-task
```

You can now test that your script is running by executing it manually:

```bash
$ /mnt/jrc-comms/hooks/boot.d/99-custom-task
DEBUG [06/06/25 16:39:27] [/mnt/jrc-comms/hooks/boot.d/99-custom-task]:      Hello World!
```

You can also test how it runs during along side the existing boot scripts by running the following command:

```bash
$ run-hooks boot
```

If all goes well, you should see your debug message printed to the console, alongside the other boot messages.

## Example: Fstab Alternative

AutoPilot's boot hook system provides a more reliable alternative to `/etc/fstab` for bind mounts within dynamic environments, particularly when dealing with paths that might not be available during early boot stages.

The following is an example of how you would implement a boot hook script to create bind mounts.
Typically this would be accomplished within `/etc/fstab`.

First, we will want to create the script in `/mnt/jrc-comms/hooks/boot.d/99-links` with the following contents:

```bash
#!/bin/bash

set -e

# Import helper functions

. /opt/jrc/lib/helpers

# Create bind mounts for shared directories

debug "creating bind mounts..."
mount --bind /var/www/domain.com/shared/var/salesexport /home/user/salesexport
mount --bind /var/www/domain.com/shared/media/importexport /home/user/importexport
```

This script will create bind mounts on boot on all instances, alternatively you can only create these mounts on specific instances by checking the instance's roles:

```bash
#!/bin/bash

set -e

# Import helper functions

. /opt/jrc/lib/helpers

# Create bind mounts for shared directories only on web and jump instances

if has_roles jump || has_roles web; then
    debug "creating bind mounts..."
    mount --bind /var/www/domain.com/shared/var/salesexport /home/user/salesexport
    mount --bind /var/www/domain.com/shared/media/importexport /home/user/importexport
else
    debug "skipping bind mounts for non-web/jump instance"
fi
```

After creating the script, ensure it has the correct permissions and ownership:

```bash
chmod +x /mnt/jrc-comms/hooks/boot.d/99-links
chown root:root /mnt/jrc-comms/hooks/boot.d/99-links
```

You can now test the script's functionality by triggering it manually or rebooting the instance.

## Troubleshooting Tips

If you want to troubleshoot the boot process, you can look at the instance's logs:

```bash
less /var/log/cloud-init-output.log
```

Alternatively, you can use the `cluster` command to view the boot logs for all the instances and even follow the logs in real-time:

```bash
cluster logs -f
```

Filtering based on role is also possible, for example, to view only the logs for the `web` role:

```bash
cluster logs -f --role web
```