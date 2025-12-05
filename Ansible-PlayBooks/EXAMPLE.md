PlayBook Examplea:
--------------------

##  Example 1: Install a Package
```yaml
- name: Install Nginx on webservers
  hosts: webservers
  become: yes
  tasks:
    - name: Ensure Nginx is installed
      yum:
        name: nginx
        state: present
```
👉 This playbook installs **Nginx** on all hosts in the `webservers` group.

---

##  Example 2: Copy a File
```yaml
- name: Deploy configuration file
  hosts: all
  become: yes
  tasks:
    - name: Copy custom config
      copy:
        src: /home/user/config.conf
        dest: /etc/myapp/config.conf
        owner: root
        group: root
        mode: '0644'
```
👉 Copies a local file to remote hosts with correct permissions.

---

##  Example 3: Start a Service
```yaml
- name: Ensure Apache is running
  hosts: webservers
  become: yes
  tasks:
    - name: Start Apache service
      service:
        name: httpd
        state: started
```
👉 Makes sure **Apache** is running on target servers.

---

## Example 4: Create a User
```yaml
- name: Create a new user
  hosts: all
  become: yes
  tasks:
    - name: Add user 'deploy'
      user:
        name: deploy
        state: present
        shell: /bin/bash
```
👉 Adds a `deploy` user across all hosts.

---

## Example 5: Using Handlers
```yaml
- name: Update Apache config
  hosts: webservers
  become: yes
  tasks:
    - name: Copy Apache config
      copy:
        src: apache.conf
        dest: /etc/httpd/conf/httpd.conf
      notify: Restart Apache

  handlers:
    - name: Restart Apache
      service:
        name: httpd
        state: restarted
```
👉 If the config changes, the handler restarts Apache.

---

## ⚡ Pro Tip for You,
Since you’re automating **repo setups and Windows onboarding**, you can combine these basics into a **multi-task playbook**:  
- Create repo files (`copy` module).  
- Install required packages (`yum`/`dnf`).  
- Configure services (`service` module).  
- Add users (`user` module).  


Would you like me to draft a **ready-to-run playbook** that sets up a CentOS repo from a DVD source (BaseOS + AppStream), installs a package, and validates it — all in one file? That would directly align with your repo automation project.
