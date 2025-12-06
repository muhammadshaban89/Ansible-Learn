
##  Example Playbook: Create a User

```yaml
- name: Create a new user on managed hosts
  hosts: all
  become: yes   # escalate privileges to root
  vars:
    username: deploy
    user_shell: /bin/bash
  tasks:
    - name: Ensure user "{{ username }}" exists
      user:
        name: "{{ username }}"
        state: present
        shell: "{{ user_shell }}"
        create_home: yes
```

###  Explanation
- **`hosts: all`** → applies to all hosts in your inventory.  
- **`become: yes`** → ensures root privileges (needed for user management).  
- **`vars:`** → defines variables (`username`, `user_shell`) for flexibility.  
- **`user:` module** → creates the user if not present, idempotently.  
- **`create_home: yes`** → ensures a home directory is created.  

---

##  Example with Password and Groups
```yaml
- name: Create user with password and groups
  hosts: webservers
  become: yes
  vars:
    username: appadmin
    user_password: "{{ 'MySecurePass123' | password_hash('sha512') }}"
  tasks:
    - name: Add user "{{ username }}" with password
      user:
        name: "{{ username }}"
        password: "{{ user_password }}"
        groups: "wheel"
        append: yes
        shell: /bin/bash
```

👉 This version:
- Uses **hashed password** (never store plain text).  
- Adds the user to the **wheel group** (sudo privileges).  
- Ensures the shell is `/bin/bash`.  

---

## Example with Host Variables
If you want **different users per host**, define them in `host_vars`:

**Inventory:**
```ini
[webservers]
web1 username=webadmin
web2 username=deploy
```

**Playbook:**
```yaml
- name: Create host-specific users
  hosts: webservers
  become: yes
  tasks:
    - name: Ensure user exists
      user:
        name: "{{ username }}"
        state: present
```

👉 Each host gets its own user based on inventory variables.

---

##  Pro Tips
- Always **hash passwords** using `password_hash`.  
- Use **group_vars** for shared defaults (e.g., default shell).  
- Test with `--check` mode before applying changes.  
- Combine with **handlers** if you need to restart services after user creation.  

---

.

---

Example-2:
---------
##  Playbook: Install httpd, firewalld, Enable Services, Add Firewall Rule
```yaml
- name: Setup Apache webserver with firewall rule
  hosts: webservers
  become: yes
  vars:
    web_service: httpd
    firewall_service: firewalld
    firewall_rules:
      - http
      - https

  tasks:
    - name: Install web service
      yum:
        name: "{{ web_service }}"
        state: present

    - name: Install firewall service
      yum:
        name: "{{ firewall_service }}"
        state: present

    - name: Ensure web service is enabled and started
      service:
        name: "{{ web_service }}"
        state: started
        enabled: yes

    - name: Ensure firewall service is enabled and started
      service:
        name: "{{ firewall_service }}"
        state: started
        enabled: yes

    - name: Allow firewall rules
      firewalld:
        service: "{{ item }}"
        permanent: yes
        state: enabled
        immediate: yes
      loop: "{{ firewall_rules }}"
```

---

##  Explanation
- **`yum` module** → installs `httpd` and `firewalld`.  
- **`service` module** → ensures both services are enabled at boot and started now.  
- **`firewalld` module** → adds firewall rules:
  - `service: http` → opens port 80.  
  - `service: https` → opens port 443.  
  - `permanent: yes` → persists across reboots.  
  - `immediate: yes` → applies instantly without reload.  

---

## ⚡ Pro Tips
- If you only want HTTP, you can remove the HTTPS task.  
- To allow a **custom port** (e.g., 8080):
  ```yaml
    - name: Allow custom port
      firewalld:
        port: 8080/tcp
        permanent: yes
        state: enabled
        immediate: yes
  ```
- Always test with:
  ```bash
  ansible-playbook setup_webserver.yml --check
  ```

---
