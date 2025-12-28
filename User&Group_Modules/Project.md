Simple_Project:
---------------

- Creates `pubkeys/` **on the control node**
- Generates SSH key pairs **on the control node** for each user
- Stores each user’s public key in `pubkeys/<username>.pub`
- **Then continues with:**  
- creating users  
- adding them to `devteam`  
- setting passwords from vault  
- creating `.ssh` directories  
- installing authorized_keys  

**Important Note**
To run tasks on the **control node**, we must use:

```yaml
delegate_to: localhost
run_once: true
```

This ensures key generation happens **only once** and **only on the controller**, not on every managed host.

---

**FINAL PLAYBOOK — Fully Automated SSH Key + User Management**

```yaml
---
- name: Manage users, groups, and SSH keys
  hosts: rhelnode
  become: yes

  vars_files:
    - vault.yml

  vars:
    user_list:
      - dev
      - tester
      - admin

  tasks:

    #####################################################################
    # 1. Create pubkeys/ directory on control node
    #####################################################################
    - name: Ensure pubkeys directory exists on control node
      file:
        path: "pubkeys"
        state: directory
        mode: '0755'
      delegate_to: localhost
      run_once: true

    #####################################################################
    # 2. Generate SSH key pairs on control node for each user
    #####################################################################
    - name: Generate SSH key pair for each user on control node
      command: >
        ssh-keygen -t rsa -b 2048 -f pubkeys/{{ item }} -N ""
      args:
        creates: "pubkeys/{{ item }}"
      delegate_to: localhost
      loop: "{{ user_list }}"
      run_once: true

    #####################################################################
    # 3. Ensure devteam group exists on managed nodes
    #####################################################################
    - name: Ensure devteam group exists
      group:
        name: devteam
        state: present

    #####################################################################
    # 4. Create users with devteam as secondary group
    #####################################################################
    - name: Create users with devteam as secondary group
      user:
        name: "{{ item }}"
        password: "{{ user_passwords[item] }}"
        groups: devteam
        append: yes
        create_home: yes
        shell: /bin/bash
      loop: "{{ user_list }}"

    #####################################################################
    # 5. Create .ssh directory for each user
    #####################################################################
    - name: Ensure .ssh directory exists
      file:
        path: "/home/{{ item }}/.ssh"
        state: directory
        owner: "{{ item }}"
        group: "{{ item }}"
        mode: '0700'
      loop: "{{ user_list }}"

    #####################################################################
    # 6. Install authorized_keys for each user
    #####################################################################
    - name: Install authorized_keys for each user
      authorized_key:
        user: "{{ item }}"
        state: present
        key: "{{ lookup('file', 'pubkeys/' + item + '.pub') }}"
      loop: "{{ user_list }}"
```
- vault.yml
```yaml
user_passwords:
  dev: "$6$HASHEDPASSWORD1"
  tester: "$6$HASHEDPASSWORD2"
  admin: "$6$HASHEDPASSWORD3"
```
- create vault.
```yaml
ansible-vault edit vault.yml
```
---

**How This Works (Step-by-Step)**

**Step 1 — Create `pubkeys/` on control node**
This ensures the folder exists before generating keys.

**Step 2 — Generate SSH keys on control node**
For each user, it creates:

```
pubkeys/dev
pubkeys/dev.pub
pubkeys/tester
pubkeys/tester.pub
pubkeys/admin
pubkeys/admin.pub
```

The `.pub` files are used later.

**Step 3 — Create group on managed nodes**
`devteam` is created once per host.

**Step 4 — Create users**
Passwords come from Vault.

**Step 5 — Create `.ssh` directories**
Correct permissions for SSH.

**Step 6 — Install authorized_keys**
Each user gets their own public key.


