---
title: Add Custom Boot Hook to /mnt/jrc-comms/hooks/boot.d
---

This guide explains how to add a custom boot hook to `/mnt/jrc-comms/hooks/boot.d/` in applications that execute scripts in numerical order. Follow these steps to ensure proper execution order and functionality.

!!! Assumption:
We assume you have root access to modify files in `/mnt/jrc-comms/hooks/boot.d/`.
!!!

## Create Custom Boot Hook

### 1. Create Your Script  
Create a new shell script with a filename starting with `99-` (to ensure it runs last). For example:  

```bash
#!/bin/bash
/mnt/jrc-comms/hooks/boot.d/99-custom-task

echo "Running custom boot task..."
```

### 2. Set Permissions and Ownership  
Make the script executable and ensure it’s owned by `root`:  

### 3. Verify Execution Order  
The application runs scripts in ascending order based on their numeric prefixes. For example:  

- `05-mount-recovery-mount` runs first  
- `10-cluster-clean` runs next  
- `99-custom-task` runs last  

**Key rules:**  
- Prefixes determine execution order (e.g., `00-` to `99-`)  
- Use two-digit numbering for clarity (e.g., `05-`, `10-`, `99-`)  
- Scripts with the same prefix may execute in lexicographical order, but avoid ambiguity by using unique prefixes  

---

## Example Walkthrough  
To log system time at the end of the boot process:  

```bash
#!/bin/bash
/mnt/jrc-comms/hooks/boot.d/99-custom-task

LOGFILE="/var/log/custom-boot.log"
echo "Boot completed at: $(date)" >> $LOGFILE
```

---

## Troubleshooting Tips  
1. **Test your script manually** before relying on the boot process: 

```bash
sudo /mnt/jrc-comms/hooks/boot.d/99-custom-task
```

2. **Check logs** for errors (e.g., `/var/log/syslog` or `journalctl`)  

3. **Validate script syntax** using:  

```bash
bash -n /mnt/jrc-comms/hooks/boot.d/99-custom-task
```

By following this structure, you can extend the boot process with custom logic while maintaining predictable execution order.

---

### Alternative to fstab Mounting Using Boot Hook Script

AutoPilot's boot hook system provides a more reliable alternative to `/etc/fstab` for bind mounts within dynamic environments, particularly when dealing with paths that might not be available during early boot stages.

The following is an example of how you would implement a boot hook script to create hard links. Typically this would be accomplished within `/etc/fstab`.

## Boot Hook Script Implementation

### 1. Create /mnt/jrc-comms/hooks/boot.d/99-links Script

Creates bind mounts after filesystems are available

```bash
#!/bin/bash
/mnt/jrc-comms/hooks/boot.d/99-links

echo "Creating bind mounts..."
mount --bind /var/www/domain.com/shared/var/salesexport /home/user/salesexport
mount --bind /var/www/domain.com/shared/media/importexport /home/user/importexport
```

### 2. Set Permissions

```bash
sudo chmod +x /mnt/jrc-comms/hooks/boot.d/99-links
sudo chown root:root /mnt/jrc-comms/hooks/boot.d/99-links
```

## Verification
1. **Validate script syntax**:

```bash
bash -n /mnt/jrc-comms/hooks/boot.d/99-custom-task
```

2. **Check active mounts**:

```bash
mount | grep bind
```

