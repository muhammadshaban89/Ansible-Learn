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

Purpose
- Transfers files from your control node (local machine) to the managed hosts.
- Can also create files with specific content directly on the target.
- Useful for distributing configs, scripts, or templates.
  
Key Parameters
- src: Path to the source file on the control node.
- dest: Destination path on the managed host.
- content: Inline text to write instead of copying a file.
- owner, group, mode: Permissions and ownership.

```yaml
- name: Copy local file to remote
  ansible.builtin.copy:
    src: /home/user/config.cfg
    dest: /etc/myapp/config.cfg
    owner: root
    mode: '0644'
```
ad hoc 
```yaml
ansible all -m copy -a "src=/home/admin/config.cfg dest=/etc/myapp/config.cfg"
```
### `file`
Purpose of the file Module:
The file module is used to set attributes of files, directories, and symlinks.
It doesn’t copy or edit contents — instead, it ensures the state (present, absent, directory, link, etc.) and attributes (permissions, ownership, etc.) are correct.

 Key Parameters
- path: The file or directory path to manage.
- state: Desired state of the path.
- touch → create an empty file if it doesn’t exist.
- directory → ensure a directory exists.
- absent → remove file/directory.
- file → ensure a regular file exists.
- link → create a symbolic link.
- owner / group: Set file ownership.
- mode: Set permissions (e.g., '0644').
- recurse: Apply ownership/permissions recursively to directories.

```yaml
- name: Ensure directory exists
  ansible.builtin.file:
    path: /var/log/myapp
    state: directory
    owner: myuser
    mode: '0755'
```

### `lineinfile`

Purpose:
- Ensures a specific line exists in a file.
- Can add, replace, or modify lines based on regex.
- Ideal for editing configuration files without replacing the whole file.
  
Key Parameters
- path: File to edit.
- line: The exact line to insert/ensure.
- regexp: Pattern to search for existing line(s).
- state: Defaults to present (ensure line exists). Can also be absent.

```yaml
- name: Ensure a line exists in a file
  ansible.builtin.lineinfile:
    path: /etc/ssh/sshd_config
    regexp: '^PermitRootLogin'
    line: 'PermitRootLogin no'
```
Ad‑hoc command:
```yaml
ansible all -m lineinfile -a "path=/etc/ssh/sshd_config regexp='^PermitRootLogin' line='PermitRootLogin no'"
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

**Thanks:**

*👉Follow my LinkdIn Profile: www.linkedin.com/in/muhammad-shaban-45577719a*

