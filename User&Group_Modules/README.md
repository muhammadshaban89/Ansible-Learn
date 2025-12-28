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


**What the `authorized_key` Module Does**

- The `authorized_key` module manages the **SSH authorized_keys file** for a user on a **managed node**.
- This is the file that controls **which public keys are allowed to SSH into that user account**.

So this module is essential when you want to:

- Add SSH keys for users  
- Remove SSH keys  
- Rotate keys  
- Manage access across many servers  
- Automate onboarding/offboarding  

**Example**

```yaml
- name: Set authorized key
  authorized_key:
    user: user1
    state: present
    key: "{{ lookup('file', '/home/user1/.ssh/id_rsa.pub') }}"
```


## `user: user1`
This tells Ansible:

> “Modify the `authorized_keys` file for the user **`user1`** on the remote server.”

The file being modified is:

```
/home/user1/.ssh/authorized_keys
```

## `state: present`
This means:

- Ensure the key **exists** in the `authorized_keys` file  
- If it’s missing → add it  
- If it’s already there → do nothing (idempotent)

If you wanted to remove a key, you would use:

```yaml
state: absent
```

## `key: "{{ lookup('file', '/home/user1/.ssh/id_rsa.pub') }}"`
This is the most important part.

### What it does:
- Reads the **public key file** from the **Ansible control node**, not the remote server  
- Inserts that key into the remote user’s authorized_keys file

## Why lookup is used:
`lookup('file', ...)` means:

> “Read the contents of this file on the controller machine.”

So Ansible takes the public key stored on your control node and pushes it to the managed node.

**Why This Module Is Important**

- Imagine you have 50 servers and 10 users.
- Manually copying SSH keys to each server is painful.

With this module, you can:

- Add a user’s key to all servers  
- Remove a user’s key from all servers  
- Rotate keys automatically  
- Enforce consistent access control  

**A More Realistic Example (Using a Loop)**

```yaml
- name: Deploy SSH keys for multiple users
  authorized_key:
    user: "{{ item.username }}"
    key: "{{ lookup('file', 'pubkeys/' + item.username + '.pub') }}"
    state: present
  loop:
    - { username: 'muhammad' }
    - { username: 'ahmed' }
    - { username: 'devops' }
```
This is how you scale SSH key management across many users and servers.




