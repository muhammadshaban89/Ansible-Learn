Schedule Future Jobs:
----------------------
- The `at` and `Cron` modules are provided by Ansible are used to schedule future jobs on managed host.

# Ansible `at` Module  
The **`at`** module schedules **one‑time tasks** on the target host.  
Perfect for delayed actions, maintenance windows, or post‑deployment cleanup.


#Basic Example — Run a command once after 5 minutes
```yaml
- name: Schedule a one-time task using at
  hosts: all
  become: yes
  tasks:
    - name: Run cleanup script after 5 minutes
      ansible.builtin.at:
        command: "/usr/local/bin/cleanup.sh"
        count: 5
        units: minutes
        unique: yes
```


# Schedule at a specific time
```yaml
- name: Run a script at 2:30 AM
  hosts: all
  become: yes
  tasks:
    - name: Schedule backup
      ansible.builtin.at:
        command: "/usr/local/bin/backup.sh"
        time: "02:30"
```

# Remove a scheduled at job
```yaml
- name: Remove an at job
  hosts: all
  become: yes
  tasks:
    - name: Delete job with ID 12
      ansible.builtin.at:
        job_id: 12
        state: absent
```
#  Ansible `cron` Module  
- The **`cron`** module manages **recurring scheduled tasks**.  
- Great for backups, log rotation, monitoring scripts, cleanup routines, etc.

# Example — Create a daily cron job
```yaml
- name: Create a daily cron job
  hosts: all
  become: yes
  tasks:
    - name: Run backup every day at 3 AM
      ansible.builtin.cron:
        name: "Daily backup"
        minute: "0"
        hour: "3"
        job: "/usr/local/bin/backup.sh"
```

# Example — Run every 15 minutes
```yaml
- name: Run script every 15 minutes
  hosts: all
  become: yes
  tasks:
    - name: Monitor disk usage
      ansible.builtin.cron:
        name: "Disk monitor"
        minute: "*/15"
        job: "/usr/local/bin/disk_check.sh"
```

# Remove a cron job
```yaml
- name: Remove a cron job
  hosts: all
  become: yes
  tasks:
    - name: Delete disk monitor job
      ansible.builtin.cron:
        name: "Disk monitor"
        state: absent
```


# Add cron job to a specific user
```yaml
- name: Add cron job for user 'muhammad'
  hosts: all
  become: yes
  tasks:
    - name: User-specific cron
      ansible.builtin.cron:
        name: "User cleanup"
        minute: "0"
        hour: "1"
        user: "muhammad"
        job: "/home/muhammad/cleanup.sh"
```

- **EXAMPLE-PLAYBOOK**
```yaml
- name: Deploy cleanup and backup scripts, then schedule cron jobs
  hosts: all
  become: yes

  tasks:
    - name: Copy clean.sh to managed host
      ansible.builtin.copy:
        src: files/clean.sh
        dest: /usr/local/bin/clean.sh
        mode: '0755'
      notify: restart_cron

    - name: Copy backup.sh to managed host
      ansible.builtin.copy:
        src: files/backup.sh
        dest: /usr/local/bin/backup.sh
        mode: '0755'
      notify: restart_cron

    - name: Create nightly cleanup cron job
      ansible.builtin.cron:
        name: "Nightly cleanup"
        minute: "0"
        hour: "2"
        job: "/usr/local/bin/clean.sh"

    - name: Create daily backup cron job
      ansible.builtin.cron:
        name: "Daily backup"
        minute: "0"
        hour: "3"
        job: "/usr/local/bin/backup.sh"

  handlers:
    - name: restart_cron
      ansible.builtin.service:
        name: crond
        state: restarted
```

- **Log and tmp cleanup script `clean.sh`** that:

-  rotates logs
- clears temp files older than X days
- logs its own actions
- never deletes blindly

```yaml
#!/bin/bash

LOGFILE="/var/log/cleanup.log"
TMP_DIR="/tmp"
DAYS=7

echo "===== Cleanup started at $(date) =====" >> "$LOGFILE"

# Remove temp files older than 7 days
find "$TMP_DIR" -type f -mtime +$DAYS -print -delete >> "$LOGFILE" 2>&1

# Rotate system logs (safe, non-destructive)
if command -v logrotate >/dev/null 2>&1; then
    logrotate -f /etc/logrotate.conf >> "$LOGFILE" 2>&1
fi

echo "Cleanup completed at $(date)" >> "$LOGFILE"
echo "" >> "$LOGFILE"
```

- **Simple Backup Scrip** `backup.sh`  --> That:
-  backs up a directory
- compresses it
- stores it with a timestamp
- logs everything
- avoids overwriting existing backups
```yaml
#!/bin/bash

SOURCE_DIR="/etc"
BACKUP_DIR="/var/backups"
LOGFILE="/var/log/backup.log"
TIMESTAMP=$(date +"%Y%m%d_%H%M%S")
ARCHIVE="$BACKUP_DIR/etc_backup_$TIMESTAMP.tar.gz"

mkdir -p "$BACKUP_DIR"

echo "===== Backup started at $(date) =====" >> "$LOGFILE"

# Create compressed archive
tar -czf "$ARCHIVE" "$SOURCE_DIR" >> "$LOGFILE" 2>&1

# Verify archive
if [ -f "$ARCHIVE" ]; then
    echo "Backup successful: $ARCHIVE" >> "$LOGFILE"
else
    echo "Backup FAILED at $(date)" >> "$LOGFILE"
fi

echo "Backup completed at $(date)" >> "$LOGFILE"
echo "" >> "$LOGFILE"
```

- **EXAMPLE: Disk_check_script. `disk_check.sh`**

```yaml
#!/bin/bash

LOGFILE="/var/log/disk_check.log"
THRESHOLD=80   # alert if usage exceeds 80%

echo "===== Disk Check started at $(date) =====" >> "$LOGFILE"

# Loop through all mounted filesystems except tmpfs, devtmpfs, squashfs
df -hP | grep -vE 'tmpfs|devtmpfs|squashfs' | tail -n +2 | while read -r line; do
    USAGE=$(echo "$line" | awk '{print $5}' | tr -d '%')
    MOUNT=$(echo "$line" | awk '{print $6}')

    if [ "$USAGE" -ge "$THRESHOLD" ]; then
        echo "WARNING: $MOUNT is at ${USAGE}% usage" >> "$LOGFILE"
    else
        echo "OK: $MOUNT is at ${USAGE}% usage" >> "$LOGFILE"
    fi
done

echo "Disk Check completed at $(date)" >> "$LOGFILE"
echo "" >> "$LOGFILE"
```

