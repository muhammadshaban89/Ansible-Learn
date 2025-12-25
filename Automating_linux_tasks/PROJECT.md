
Complete Ansible Project for Automating Linux Administration
------------------------------------------------------------

##  **Project Structure**
```yaml
simple_project/
├── site.yml
├── inventory/
│   └── hosts
├── group_vars/
│   └── all.yml
├── roles/
│   ├── users/
│   │   ├── tasks/main.yml
│   │   └── defaults/main.yml
│   ├── packages/
│   │   └── tasks/main.yml
│   ├── services/
│   │   └── tasks/main.yml
│   ├── files/
│   │   ├── tasks/main.yml
│   │   └── files/index.html
│   ├── cron/
│   │   └── tasks/main.yml
|   |   └── files/clean.sh
│   ├── timesync/
│   │   └── tasks/main.yml
│   ├── backup/
│   │   ├── tasks/main.yml
│   │   └── files/backup.sh
│   └── logrotate/
│       ├── tasks/main.yml
│       └── files/myapp.logrotate
```

---

# **1. Main Orchestration Playbook (site.yml)**

```yaml
- name: Automate Linux Administration Tasks
  hosts: all
  become: yes

  roles:
    - users
    - packages
    - services
    - files
    - cron
    - timesync
    - backup
    - logrotate
```

This playbook calls all roles in order.

---

# **2. Users Role (roles/users/tasks/main.yml)**

```yaml
- name: Create admin users
  user:
    name: "{{ item.name }}"
    shell: "{{ item.shell }}"
    state: present
  loop:
    - { name: "devops", shell: "/bin/bash" }
    - { name: "tester", shell: "/bin/bash" }
```

---

#  **3. Packages Role (roles/packages/tasks/main.yml)**

```yaml
- name: Install common packages
  package:
    name:
      - vim
      - tree
      - wget
      - net-tools
      - nginx
    state: present
```

---

#  **4. Services Role (roles/services/tasks/main.yml)**

```yaml
- name: Ensure nginx is running
  service:
    name: nginx
    state: started
    enabled: yes
```

---

#  **5. Files Role (roles/files/tasks/main.yml)**

```yaml
- name: Deploy index page
  copy:
    src: index.html
    dest: /usr/share/nginx/html/index.html
    mode: '0644'
```

Place `index.html` inside:

    roles/files/files/index.html
```yaml
<!DOCTYPE html>
<html>
<head>
    <title>Simple Ansible Project</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #fafafa;
            text-align: center;
            padding-top: 80px;
        }
        h1 {
            color: #222;
        }
        p {
            color: #555;
            font-size: 18px;
            margin: 10px 0;
        }
        a {
            color: #0077b5;
            text-decoration: none;
            font-weight: bold;
        }
        a:hover {
            text-decoration: underline;
        }
    </style>
</head>
<body>
    <h1>Welcome!</h1>
    <p>This is a simple Ansible project designed to automate Linux administration tasks.</p>
    <p>Connect with me on LinkedIn:</p>
    <p><a href="https://www.linkedin.com/in/muhammad-shaban-45577719a" target="_blank">
        www.linkedin.com/in/muhammad-shaban-45577719a
    </a></p>
    <p>Explore my Ansible Study Guide for Beginners:</p>
    <p><a href="https://github.com/muhammadshaban89/Ansible-Learn" target="_blank">
        github.com/muhammadshaban89/Ansible-Learn
    </a></p>
</body>
</html>
```

---

#  **6. Cron Role (roles/cron/tasks/main.yml)**

```yaml
- name: Copy cleanup script to target node
  copy:
    src: clean.sh
    dest: /usr/local/bin/cleanup.sh
    mode: '0755'

- name: Setup daily cleanup cron job
  cron:
    name: "cleanup logs"
    job: "/usr/local/bin/cleanup.sh"
    minute: "0"
    hour: "2"
```
- **clean.sh**:

      roles/cron/files/clean.sh
```bash
#!/bin/bash

# Cleanup temporary files older than 7 days
find /tmp -type f -mtime +7 -delete

# Cleanup old logs in /var/log/myapp
find /var/log/myapp -type f -name "*.log" -mtime +7 -delete

# Cleanup system cache (optional)
rm -rf /var/cache/yum/*
```
---

#  **7. Time Sync Role (roles/timesync/tasks/main.yml)**

```yaml
- name: Configure NTP
  include_role:
    name: rhel-system-roles.timesync

  vars:
    timesync_ntp_servers:
      - hostname: 0.asia.pool.ntp.org
      - hostname: 1.asia.pool.ntp.org
      - hostname: 2.asia.pool.ntp.org
```
#  **8.Backup_Automation roles/backup/tasks/main.yml**
```yaml
- name: Copy backup script
  copy:
    src: backup.sh
    dest: /usr/local/bin/backup.sh
    mode: '0755'

- name: Create backup directory
  file:
    path: /var/backups/system
    state: directory
    mode: '0755'

- name: Schedule daily backup at 1 AM
  cron:
    name: "Daily system backup"
    job: "/usr/local/bin/backup.sh"
    minute: "0"
    hour: "1"
```
- backup.sh
```yaml
#!/bin/bash
BACKUP_DIR="/var/backups/system"
mkdir -p $BACKUP_DIR

tar -czf $BACKUP_DIR/etc-backup-$(date +%F).tar.gz /etc
find $BACKUP_DIR -type f -mtime +7 -delete
```
- **what does it will do?**

✔ Creates backup directory

✔ Compresses /etc

✔ Keeps backups for 7 days


 #  **8 Log_Rotate roles/logrotate/tasks/main.yml**
 ```yaml
- name: Deploy logrotate configuration
  copy:
    src: myapp.logrotate
    dest: /etc/logrotate.d/myapp
    mode: '0644'
```
- **roles/logrotate/files/myapp.logrotate**
```yaml
# Rotates logs daily
# Keeps 7 days
# Compresses old logs

/var/log/myapp/*.log {
    daily
    rotate 7
    compress
    missingok
    notifempty
    create 0640 root root
}
```

---

# **What This Automation Covers**
This project automates:

### ✔ User creation  
### ✔ Package installation  
### ✔ Service management  
### ✔ File deployment  
### ✔ Cron jobs  
### ✔ Time synchronization  
### ✔ NGINX deployment  
### ✔ Enterprise‑grade system roles 
### ✔ Backup Automation
### ✔ log_Rotate

