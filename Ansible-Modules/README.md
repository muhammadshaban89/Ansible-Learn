Overview:
---------

**Ansible modules are the core building blocks of Ansible automation — small programs that perform specific tasks like installing packages, managing files, configuring services, or interacting with APIs.**  

---

###  Key Points About Ansible Modules
- **Definition**: An Ansible module is a **reusable, standalone script** (usually written in Python) that executes a particular task on a target system.  
- **Role in Ansible**: Modules are the **units of work** in Ansible. Every playbook task calls a module to perform an action.  
- **Execution**: They run on the managed nodes (remote hosts) and return JSON output to Ansible, which interprets success or failure.  
- **Variety**: There are **thousands of modules** available in Ansible’s collections (system, cloud, networking, database, Windows, etc.).  
- **Customization**: You can also write **custom modules** for specialized tasks and share them with your team or community.  

---

###  Categories of Ansible Modules
Here are some common module types you’ll encounter:

| **Category**          | **Examples**                                                                 |
|------------------------|------------------------------------------------------------------------------|
| **System**             | `user`, `group`, `service`, `systemd`, `cron`                               |
| **Package Management** | `yum`, `apt`, `dnf`, `pip`, `npm`                                           |
| **Files**              | `copy`, `template`, `file`, `lineinfile`, `blockinfile`                      |
| **Database**           | `mysql_db`, `postgresql_db`, `mongodb`                                      |
| **Cloud**              | `ec2` (AWS), `azure_rm` (Azure), `gcp_compute` (Google Cloud)               |
| **Networking**         | `ios_command`, `junos_config`, `nxos_facts`                                 |
| **Windows**            | `win_service`, `win_package`, `win_file`                                    |

 

---

###  Example Usage


```yaml
- name: Install Apache on Debian
  hosts: webservers
  tasks:
    - name: Ensure Apache is installed
      apt:
        name: apache2
        state: present

    - name: Start Apache service
      service:
        name: apache2
        state: started
```

- The **`apt` module** installs Apache.  
- The **`service` module** ensures Apache is running.  

---

### Why They Matter

- **Consistency**: Modules enforce desired state across multiple systems.  
- **Efficiency**: Automate repetitive tasks without writing custom scripts.  
- **Extensibility**: Build your own modules for unique infrastructure needs.
  
------------------------------------------------------------------------------

- The `ansible-doc -l` command lists all modules installed on a system.
-  You can use `ansible- doc` to view the documentation of particular modules by name, and find information about what 
arguments the modules take as options. For example, the following command displays 
documentation for the ping module:
```
[user@controlnode ~]$ ansible-doc ping
```
- Most modules are idempotent, which means that they can be run safely multiple times, and if the 
system is already in the correct state, they do nothing.


Commond,shell,raw modules:
--------------------------
- The `command` module allows administrators to run arbitrary commands on the command line of 
managed hosts.
- The command to be run is specified as an argument to the module using the -a 
option.
```
[user@controlnode ~]$ ansible mymanagedhosts -m command -a /usr/bin/hostname
```
- The `command` module allows administrators to quickly execute remote commands on managed 
hosts.
- These commands are not processed by the shell on the managed hosts. As such, they 
cannot access shell environment variables or perform shell operations, such as redirection and 
piping.
- For situations where commands require shell processing, administrators can use the `shell` 
module.
- Like the `command` module, you pass the commands to be executed as arguments to 
the module in an ad hoc command.
- Ansible then executes the command remotely on the managed hosts.
- Unlike the `command` module, the commands are processed through a shell on the managed hosts. Therefore, shell environment variables are accessible and shell operations such as 
redirection and piping are also available for use.

**EXAMPLE**
```
[user@controlnode ~]$ ansible localhost -m command -a set
[user@controlnode ~]$ ansible localhost -m shell -a set
```

**raw**

- Both command and shell modules require a working Python installation on the managed host. 
- A third module, `raw`, can run commands directly using the remote shell, bypassing the module subsystem.
- This is useful when managing systems that cannot have Python installed (for example, 
a network router).It can also be used to install Python on a host.

Note:
----- 
**In most circumstances, it is a recommended practice that you avoid the `command, 
shell, and raw` "run command" modules. 
Most other modules are idempotent and can perform change tracking automatically. 
They can test the state of systems and do nothing if those systems are already in 
the correct state. By contrast, it is much more complicated to use "run command" 
modules in a way that is idempotent. Depending upon them makes it harder for you 
to be confident that rerunning an ad hoc command or playbook would not cause an 
unexpected failure. When a shell or command module runs, it typically reports a 
CHANGED status based on whether it thinks it affected machine state. 
There are times when "run command" modules are valuable tools and a good 
solution to a problem. If you do need to use them, it is probably best to try to use 
the command module first, resorting to shell or raw modules only if you need their 
special features*
