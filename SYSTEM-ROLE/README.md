ANSIBLE_SYSTEM_ROLES:
---------------------

- **Ansible System Roles** are a special collection of **pre‑built, supported, and standardized roles** designed to configure Linux systems consistently.
- **They’re especially common on **RHEL, CentOS Stream, Rocky, AlmaLinux**, and other Enterprise Linux systems.**
- Beginning with Red Hat Enterprise Linux 7.4, a number of Ansible roles have been provided with the operating system as part of the `rhel-system-roles` package.
- In Red Hat Enterprise Linux 8 the package is available in the `AppStream channel.`
- As an example, the recommended time synchronization service for Red Hat Enterprise Linux 7 is the chronyd service.
- In Red Hat Enterprise Linux 6 however, the recommended service is the ntpd service.
- In an environment with a mixture of Red Hat Enterprise Linux 6 and 7 hosts, an administrator must manage the configuration files for both services.
- With RHEL System Roles, administrators no longer need to maintain configuration files for both services.
- Administrators can use rhel-system-roles.timesync role to configure time synchronization for both Red Hat Enterprise Linux 6 and 7 hosts.
- RHEL System Roles are derived from the open source Linux System Roles project, found on `Ansible Galaxy`.


---

# What Are Ansible System Roles?
Ansible System Roles are **ready‑made roles** created by Red Hat to automate common system administration tasks.  
They follow best practices and work across multiple RHEL‑based distributions.

They save you from writing roles from scratch for things like:
- Network configuration  
- Storage setup  
- Time synchronization  
- SELinux  
- Firewall  
- Logging  
- Kernel settings  
- Postfix  
- Kdump  
- Certificate management  

They are **modular**, **well‑tested**, and **enterprise‑grade**.

---

# Where Do They Come From?
On RHEL‑based systems, they are installed via:

```bash
sudo yum install rhel-system-roles
```

On Rocky/AlmaLinux, the package is usually:

```bash
sudo yum install linux-system-roles
```

This installs roles under:

```
/usr/share/ansible/roles/
```
- The default `roles_path on Red` Hat Enterprise Linux includes `/usr/share/ansible/roles` in the path, so Ansible should automatically find those roles when referenced by a playbook.
- Ansible might not find the system roles if `roles_path` has been overridden in the current Ansible configuration file, if the environment variable `ANSIBLE_ROLES_PATH` is set, or if there is another role of the same name in a directory listed earlier in `roles_path`.
- After installation, documentation for the RHEL System Roles is found in the `/usr/share/doc/ rhel-system-roles-<version>/` directory. Documentation is organized into subdirectoriesby 
subsystem.

```yaml
 ls -l /usr/share/doc/rhel-system-roles/ 
```
- Each role's documentation directory contains a `README.md` file. The README.md file contains a description of the role, along with role usage information. 
- The `README.md` file also describes role variables that affect the behavior of the role. Often the `README.md` file contains a playbook snippet that demonstrates variable settings for a common configuration scenario.
- The `ansible-galaxy`command line tool  is used to manage Ansible roles, including the creation of new roles. You can run `ansible-galaxy init role-name` tocreate the directory structure for a new role.
- Once you have created the directory structure, you must write the content of the role. A good place to start is the `ROLENAME/tasks/main.yml` task file, the main list of tasks run by the role.
- Role dependencies allow a role to include other roles as dependencies. For example, a role that defines a documentation server may depend upon another role that installs and configures a web server. Dependencies are defined in the `meta/main.yml` file in the role directory hierarchy.
```yaml
--- 
dependencies:
  - role: apache 
    port: 8080
- role: postgres 
  dbname: serverlist 
  admin_user: felix
```
- **You can change behavior or role wit variables.**
-  Almost any other variable will override a role's default variables: inventory variables, play vars, inline role parameters, and so on.
- Fewer variables can override variables defined in a role's vars directory. `Facts,` variables loaded with `include_vars`, registered variables, and role parameters 
are some variables that can do that.
- Inventory variables and play vars cannot. This is important because it helps keep your play from accidentally changing the internal functioning of the role. 
- However, variables declared inline as `role parameters`,  have very high precedence. They can override variables defined in a `role's vars` directory. If a role parameter has the same name as a variable set in play vars, a role's vars, or an inventory or playbook variable, the role parameter overrides the other variable.

```yaml
- name: use motd role playbook 
  hosts: remote.example.com 
  remote_user: devops 
  become: true 
  roles:
   - role: motd 
     system_owner: someone@host.example.com
```
- The ansible-galaxy search subcommand searches Ansible Galaxy for roles.
  ```yaml
   ansible-galaxy search 'redis' --platforms EL
  ```
- The ansible-galaxy info subcommand displays more detailed information about a role.
```yaml
 ansible-galaxy info geerlingguy.redis 
```
- The ansible-galaxy install subcommand downloads a role from Ansible Galaxy, then installs it locally on the control node.
- By default, roles are installed into the first directory that is writable in the user's roles_path.
- Based on the default `roles_path` set for Ansible, normally the role will be installed into the user's `~/.ansible/roles` directory.
- The `default roles_path` might be overridden by your current Ansible configuration file or by the environment variable `ANSIBLE_ROLES_PATH`, which affects the behavior 
of ansible-galaxy.
- You can also specify a specific directory to install the role into by using the `-p` DIRECTORY option.
  ```yaml
  ansible-galaxy install geerlingguy.redis -p roles/
  ```
- You can also use ansible-galaxy to install a list of roles based on definitions in a text file.
- For example, if you have a playbook that needs to have specific roles installed, you can create a `roles/requirements.yml` file in the project directory that specifies which roles are needed.
```yaml
- src: geerlingguy.redis 
version: "1.5.0" 
```
- *You should specify the version of the role in your requirements.yml file, especially 
for playbooks in production.*
 
- *If you do not specify a version, you will get the latest version of the role. If the 
upstream author makes changes to the role that are incompatible with your 
playbook, it may cause an automation failure or other problems*
- o install the roles using a role file, use the `-r REQUIREMENTS-FILE` option.
```yaml
ansible-galaxy install -r roles/requirements.yml
```
- The `src` keyword specifies the Ansible Galaxy role name. If the role is not hosted on Ansible Galaxy, the src keyword indicates the role's URL.
- If the role is hosted in a source control repository, the `scm` attribute is required.
- The ansible- galaxy command is capable of downloading and installing roles from either a Git-based or 
mercurial-based software repository.
- A Git-based repository requires an scm value of git, while a role hosted on a mercurial repository requires a value of `hg`.
- If the role is hosted on Ansible Galaxy or as a tar archive on a web server, the scm keyword is omitted.
- The `ansible-galaxy list` command can also manage local roles, such as those roles found in the roles directory of a playbook project.
- A role can be removed locally with the ansible-galaxy remove subcommand. `ansible-galaxy remove nginx-acme-ssh`.

---

#  Common System Roles
Here are the most widely used ones:

| Role Name | Purpose |
|-----------|---------|
| **network** | Configure interfaces, bonds, VLANs, bridges |
| **timesync** | Configure NTP/chrony |
| **selinux** | Manage SELinux modes and policies |
| **firewall** | Configure firewalld rules |
| **storage** | Manage disks, partitions, LVM, filesystems |
| **logging** | Configure rsyslog |
| **postfix** | Configure mail relay |
| **kernel_settings** | Tune sysctl and kernel parameters |
| **certificate** | Manage TLS certificates |
| **kdump** | Configure kdump crash recovery |

These roles are extremely powerful and save hours of manual YAML writing.

---

# Example: Using the `timesync` System Role
```yaml
- name: Time Synchronization Play
  hosts: rhelnode
  become: yes

  vars:
    timesync_ntp_servers:
      - hostname: 0.rhel.pool.ntp.org
        iburst: yes
      - hostname: 1.rhel.pool.ntp.org
        iburst: yes
      - hostname: 2.rhel.pool.ntp.org
        iburst: yes

    timezone: UTC

  roles:
    - rhel-system-roles.timesync

  tasks:
    - name: Set timezone
      timezone:
        name: "{{ timezone }}"
```

Variables go in `vars:`:

```yaml
timesync_ntp_servers:
  - hostname: 0.pool.ntp.org
  - hostname: 1.pool.ntp.org
```

---

#  Example: Using the `network` System Role
```yaml
- name: Configure network interface
  hosts: all
  become: yes
  roles:
    - rhel-system-roles.network
  vars:
    network_connections:
      - name: ens33
        type: ethernet
        ip:
          address:
            - 192.168.1.10/24
          gateway: 192.168.1.1
          dns:
            - 8.8.8.8
```

