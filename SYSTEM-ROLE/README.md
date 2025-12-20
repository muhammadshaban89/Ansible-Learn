ANSIBLE_SYSTEM_ROLES:
---------------------

- **Ansible System Roles** are a special collection of **pre‑built, supported, and standardized roles** designed to configure Linux systems consistently.
- **They’re especially common on **RHEL, CentOS Stream, Rocky, AlmaLinux**, and other Enterprise Linux systems.**


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
- name: Configure NTP
  hosts: all
  become: yes
  roles:
    - rhel-system-roles.timesync
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

---

#  Why System Roles Matter (Especially for You)
Given your lab automation goals, system roles help you:

- Configure **networking** consistently across nodes  
- Set up **storage** for repo mounts or Docker/K8s  
- Ensure **time sync** for cluster consistency  
- Manage **SELinux** cleanly  
- Apply **kernel tuning** for Kubernetes or performance labs  

They remove the need to reinvent the wheel.

---


Just tell me and I’ll assemble the full structure.
