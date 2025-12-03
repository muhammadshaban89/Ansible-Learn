OverView:
------
**Some importent modules like `command` , `shell`, Files Modules and software package modules are examplained hare with examples.**


---

## 1. `command` Module
- Runs a command **without a shell** (no pipes, redirects, or environment expansion).
- Safer, but limited.

```yaml
- name: Run uptime using command
  ansible.builtin.command: uptime
```

Ad‑hoc:
```bash
ansible all -m command -a "uptime"
```

---

## 2. `shell` Module
- Runs commands **through a shell** (`/bin/sh` by default).
- Supports pipes, redirects, environment variables.

```yaml
- name: Run a shell command with pipe
  ansible.builtin.shell: "cat /etc/passwd | grep root"
```

Ad‑hoc:
```bash
ansible all -m shell -a "echo $HOME"
```

---

##  3. `raw` Module
- Sends raw commands directly over SSH.
- Useful for bootstrapping systems **without Python installed**.

```yaml
- name: Install Python on a minimal system
  ansible.builtin.raw: "apt-get install -y python3"
```

Ad‑hoc:
```bash
ansible all -m raw -a "yum install -y python3"
```

---

## 4. File Modules
These manage files, directories, and symlinks.

### `copy`
```yaml
- name: Copy local file to remote
  ansible.builtin.copy:
    src: /home/user/config.cfg
    dest: /etc/myapp/config.cfg
    owner: root
    mode: '0644'
```

### `file`
```yaml
- name: Ensure directory exists
  ansible.builtin.file:
    path: /var/log/myapp
    state: directory
    owner: myuser
    mode: '0755'
```

### `lineinfile`
```yaml
- name: Ensure a line exists in a file
  ansible.builtin.lineinfile:
    path: /etc/ssh/sshd_config
    regexp: '^PermitRootLogin'
    line: 'PermitRootLogin no'
```

---

##  5. Software Package Modules
These abstract package managers (`apt`, `yum`, `dnf`, `zypper`, etc.).

### `apt` (Debian/Ubuntu)
```yaml
- name: Install nginx on Debian
  ansible.builtin.apt:
    name: nginx
    state: present
    update_cache: yes
```

### `yum` (RHEL/CentOS)
```yaml
- name: Install httpd on CentOS
  ansible.builtin.yum:
    name: httpd
    state: latest
```

### `package` (Generic abstraction)
```yaml
- name: Install curl using generic package module
  ansible.builtin.package:
    name: curl
    state: present
```

Ad‑hoc:
```bash
ansible all -m package -a "name=curl state=present"
```

---

##  Imperative Examples Summary

| Module   | Example Command | Use Case |
|----------|-----------------|----------|
| `command` | `ansible all -m command -a "uptime"` | Simple, safe commands |
| `shell`   | `ansible all -m shell -a "echo $HOME"` | Commands needing shell features |
| `raw`     | `ansible all -m raw -a "yum install -y python3"` | Bootstrapping systems without Python |
| `copy`    | Copy config file to `/etc/myapp/` | File transfer |
| `file`    | Ensure `/var/log/myapp` exists | Directory management |
| `lineinfile` | Add/modify line in `sshd_config` | Config editing |
| `apt/yum/package` | Install `nginx`, `httpd`, `curl` | Software installation |

---

**Thanks :**

**- Follow my LinkdIn Profile: www.linkedin.com/in/muhammad-shaban-45577719a*

👉 Do you want me to bundle these into a **ready-to-run playbook** that covers all modules in one YAML file for lab validation?
