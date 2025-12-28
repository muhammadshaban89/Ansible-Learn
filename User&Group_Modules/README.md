User and Group Management:
-------------------------

**THE USER MODULE**

The **Ansible `user` module** manages user accounts on **remote managed hosts**.  
It can:

- create users  
- delete users  
- set home directories  
- set shells  
- assign groups  
- set UIDs  
- manage passwords  
- generate SSH keys  

This module is essential for Linux administration automation.


**Key Concepts**

## `name` (required)
The username to create or manage.

```yaml
name: devops_user
```

##  `shell`
Sets the user’s login shell.

```yaml
shell: /bin/bash
```

##  `groups`
List of secondary groups.

```yaml
groups: sys_admins, developers
```

##  `append: yes`
Prevents overwriting existing groups.

Without `append: yes`, Ansible **replaces** all secondary groups.

##  `password`
Must be a **hashed** password (not plain text).

##  `generate_ssh_key`
Automatically creates an SSH key pair for the user.

```yaml
generate_ssh_key: yes
ssh_key_bits: 2048
ssh_key_file: .ssh/id_my_rsa
```

This will **not overwrite** an existing key.

---

 **Example: Creating a User with Groups**

```yaml
- name: Add new user to the development machine
  user:
    name: devops_user
    shell: /bin/bash
    groups: sys_admins, developers
    append: yes
```

 **Example: Create User + Generate SSH Key**

```yaml
- name: Create a SSH key for user1
  user:
    name: user1
    generate_ssh_key: yes
    ssh_key_bits: 2048
    ssh_key_file: .ssh/id_my_rsa
```


 **Common Parameters**

| Parameter | Meaning |
|----------|---------|
| `comment` | Description of the user |
| `group` | Primary group |
| `groups` | Secondary groups |
| `home` | Home directory path |
| `create_home` | Create home directory (yes/no) |
| `system` | Create a system account |
| `uid` | Set user UID |
| `state` | present/absent |

---

 **THE GROUP MODULE **

The **`group` module** manages groups on remote hosts.

It can:

- create groups  
- delete groups  
- modify groups  

---

 **Example: Ensure a Group Exists**

```yaml
- name: Verify that auditors group exists
  group:
    name: auditors
    state: present
```

 **Common Group Module Parameters**

| Parameter | Meaning |
|----------|---------|
| `name` | Group name |
| `gid` | Group ID |
| `system` | Create a system group |
| `state` | present/absent |


 **How These Modules Work Together**

In real automation, you often:

1. **Create a group**
2. **Create a user**
3. **Assign the user to the group**
4. **Generate SSH keys**
5. **Deploy authorized keys**
6. **Set permissions**


