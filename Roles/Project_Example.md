multi-role project skeleton 
---------------------------
- Create Local Repo.
- Create a sample webserver.
- Create Users.

 ***Use  BM Command to create role like  repo_setup,nginx,users **
```yaml
ansible-galaxy init role_name
```


## Project Layout
```
ansible-project/
├── site.yml                # Main orchestration playbook
├── inventory/hosts          # Inventory file
└── roles/
    ├── repo_setup/
    │   ├── tasks/main.yml
    │   ├── vars/main.yml
        ├── templates/repo.j2
    │   └── defaults/main.yml
    ├── nginx/
    │   ├── tasks/main.yml
    │   ├── handlers/main.yml
    │   ├── templates/nginx.conf.j2
    │   └── defaults/main.yml
    └── users/
        ├── tasks/main.yml
        └── defaults/main.yml
```

---

## Main Orchestration Playbook (`site.yml`)
```yaml
---
- name: Prepare all servers
  hosts: all
  become: yes
  roles:
    - repo_setup
    - users

- name: Configure web servers
  hosts: webservers
  become: yes
  roles:
    - nginx
```

---

##  Role Examples

### `roles/repo_setup/templates/repo.j2`
```yaml
name=Local AppStream
baseurl=file:///myrepo/AppStream
enabled=1
gpgcheck=0

# /etc/yum.repos.d/local-BaseOS.repo
[local-BaseOS]
name=Local BaseOS
baseurl=file:///myrepo/BaseOS
enabled=1
gpgcheck=0
```
### `roles/repo_setup/tasks/main.yml`
```yaml
- name: Configure local repo
  template:
    src: repo.j2
    dest: /etc/yum.repos.d/local.repo
```

---
### `roles/nginx/templates/nginx.conf.j2`
```yaml
server {
    listen       80;
    server_name  localhost;

    location / {
        root   /newsite;
        index  index.html index.html;
    }
}

```
### `roles/nginx/tasks/main.yml`
```yaml
- name: Install NGINX
  package:
    name: nginx
    state: present

- name: Deploy nginx.conf
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
  notify: Restart nginx
```

### `roles/nginx/handlers/main.yml`
```yaml
- name: Restart nginx
  service:
    name: nginx
    state: restarted
    enabled: yes
```

---

### `roles/users/tasks/main.yml`
```yaml
- name: Create lab users
  user:
    name: "{{ item.name }}"
    shell: "{{ item.shell }}"
    state: present
  loop: "{{ lab_users }}"
```

### `roles/users/defaults/main.yml`
```yaml
lab_users:
  - { name: "devops", shell: "/bin/bash" }
  - { name: "tester", shell: "/bin/bash" }
```

---

##  Why This Skeleton Works
- **Modularity:** Each role handles one responsibility (repos, nginx, users).  
- **Reusability:** You can reuse `repo_setup` or `users` in other projects.  
- **Scalability:** Add more roles (e.g., `db`, `monitoring`) without bloating the main playbook.  
- **Clarity:** `site.yml` just orchestrates roles, keeping it readable.

---
