

A structured breakdown of the **most common Linux admin tasks** and how to automate each one using Ansible.

---

# 1. **User Management**
### Create users, set shells, assign groups

```yaml
- name: Manage users
  hosts: all
  become: yes

  tasks:
    - name: Create users
      user:
        name: "{{ item.name }}"
        shell: "{{ item.shell }}"
        state: present
      loop:
        - { name: "devops", shell: "/bin/bash" }
        - { name: "tester", shell: "/bin/bash" }
```

---

# 2. **Package Installation**
### Install, remove, or update packages

```yaml
- name: Install packages
  hosts: all
  become: yes

  tasks:
    - name: Install nginx
      package:
        name: nginx
        state: present
```

---

# 3. **Service Management**
### Start, stop, enable services

```yaml
- name: Manage services
  hosts: all
  become: yes

  tasks:
    - name: Ensure nginx is running
      service:
        name: nginx
        state: started
        enabled: yes
```

---

#  4. **File & Directory Management**
### Create directories, set permissions

```yaml
- name: Create directory
  hosts: all
  become: yes

  tasks:
    - name: Create /opt/data
      file:
        path: /opt/data
        state: directory
        mode: '0755'
```

---

# 5. **Copy Files**
### Deploy configs, scripts, templates

```yaml
- name: Deploy config file
  hosts: all
  become: yes

  tasks:
    - name: Copy nginx config
      copy:
        src: files/nginx.conf
        dest: /etc/nginx/nginx.conf
      notify: Restart nginx

  handlers:
    - name: Restart nginx
      service:
        name: nginx
        state: restarted
```

---

# 6. **Template Configuration Files**
### Use Jinja2 templates for dynamic configs

```yaml
- name: Deploy template
  hosts: all
  become: yes

  tasks:
    - name: Deploy nginx template
      template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
```

---

# 7. **Cron Jobs**
### Automate scheduled tasks

```yaml
- name: Setup cron job
  hosts: all
  become: yes

  tasks:
    - name: Run cleanup script daily
      cron:
        name: "cleanup"
        job: "/usr/local/bin/cleanup.sh"
        minute: "0"
        hour: "2"
```

---

#8. **Firewall Management**
### Using system roles (best practice)

```yaml
- name: Configure firewall
  hosts: all
  become: yes
  roles:
    - rhel-system-roles.firewall

  vars:
    firewall:
      - service: http
        state: enabled
```

---

#9. **Time Synchronization**
### Using system roles (enterprise-grade)

```yaml
- name: Configure NTP
  hosts: all
  become: yes
  roles:
    - rhel-system-roles.timesync

  vars:
    timesync_ntp_servers:
      - hostname: 0.asia.pool.ntp.org
      - hostname: 1.asia.pool.ntp.org
```

---

# 10. **Storage & Filesystems**
### Create partitions, LVM, mount points

```yaml
- name: Create filesystem
  hosts: all
  become: yes

  tasks:
    - name: Create ext4 filesystem
      filesystem:
        fstype: ext4
        dev: /dev/sdb1

    - name: Mount filesystem
      mount:
        path: /data
        src: /dev/sdb1
        fstype: ext4
        state: mounted
```

---

# 11. **SELinux Management**
### Using system roles

```yaml
- name: Configure SELinux
  hosts: all
  become: yes
  roles:
    - rhel-system-roles.selinux

  vars:
    selinux_state: enforcing
```

---

# 12. **Full Automation Example (Your Style)**
Here’s a combined playbook automating multiple admin tasks:

```yaml
- name: Automate Linux Administration
  hosts: all
  become: yes

  roles:
    - rhel-system-roles.timesync
    - rhel-system-roles.selinux

  vars:
    timesync_ntp_servers:
      - hostname: 0.asia.pool.ntp.org
      - hostname: 1.asia.pool.ntp.org
    selinux_state: enforcing

  tasks:
    - name: Install packages
      package:
        name:
          - vim
          - tree
          - nginx
        state: present

    - name: Create users
      user:
        name: "{{ item }}"
        state: present
      loop:
        - devops
        - tester

    - name: Deploy index page
      copy:
        src: files/index.html
        dest: /usr/share/nginx/html/index.html

    - name: Ensure nginx is running
      service:
        name: nginx
        state: started
        enabled: yes
```

Thanks:

👉Follow my LinkdIn Profile: www.linkedin.com/in/muhammad-shaban-45577719a
