TROUBLESHOOTING PLAYBOOKS
-------------------------
- By default, Red Hat Ansible Engine is not configured to log its output to any log file.
- It provides a built-in logging infrastructure that can be configured through the `log_path` parameter in the default section of the ansible.cfg configuration file, or through the `$ANSIBLE_LOG_PATH` 
environment variable.
- If any or both are configured, Ansible stores output from both the `ansible` and `ansible-playbook` commands in the log file configured, either through the `ansible.cfg` configuration file or the `$ANSIBLE_LOG_PATH` environment variable. 
- If you configure Ansible to write log files to `/var/log`, then Red Hat recommends that you configure `logrotate` to manage the Ansible log files.

THE DEBUG MODULE
----------------

- The `debug` module provides insight into what is happening in the play.
- This module can display the value for a certain variable at a certain point in the play.
- This feature is key to debugging tasks that use variables to communicate with each other.
- **The debug module is used to:**
- Print variables.
- Print custom messages.
- Show host facts.
- Display task results.
- Help troubleshoot playbooks.
  
- 1
  ```yaml
  - name: Show custom message with variable
    debug:
      msg: "The server {{ inventory_hostname }} belongs to {{ ansible_os_family }} family"
  ```
- 2
```yaml
- name: Show multiple variables
  debug:
    msg:
      - "Hostname: {{ inventory_hostname }}"
      - "OS: {{ ansible_distribution }}"
      - "Version: {{ ansible_distribution_version }}"
```

MANAGING ERRORS:
---------------

- There are several issues than can occur during a playbook run, mainly related to the syntax of either the playbook or any of the templates it uses, or due to connectivity issues with the managed 
hosts (for example, an error in the host name of the managed host in the inventory file).
- Those errors are issued by the `ansible-playbook` command at execution time.

- **1: use `--syntax-check`**
  ```yaml
  ansible-playbook site.yml --syntax-check
  ```
 ✔ What it catches
- Bad indentation
- Missing colons
- Invalid YAML
- Invalid module arguments
- Missing roles or tasks

 - **2: use `--step`**:
   
- You can also use the `--step` option to step through a playbook one task at a time.
- The `ansible- playbook --step` command interactively prompts for confirmation that you want each task to run.
  ```yaml
   ansible-playbook play.yml --syntax-check
  ```
- Useful for
- Debugging
- Learning how a playbook flows
- Testing tasks one by one
  
**2: use `--start-at-task`**:

- The `--start-at-task` option allows you to start execution of a playbook from a specific task. It takes as an argument the name of the task at which to start.
  ```yaml
  ansible-playbook site.yml --start-at-task="Install NGINX"
  ```
- You can increase verbosity of `ansible-playbook` command by adding one or more `-v` switch:
   **Verbosity Options**

| Option | Description |
|--------|-------------|
| **-v** | Shows basic output. Displays task results in more detail than default. |
| **-vv** | Shows both **input and output** data. Useful for seeing module arguments. |
| **-vvv** | Adds **connection debugging**. Shows SSH details, control socket info, and connection negotiation. |
| **-vvvv** | Adds **deep debugging**. Shows scripts executed on remote hosts, environment variables, and the user executing each script. Also shows internal Ansible operations. |

---
RECOMMENDED PRACTICES FOR PLAYBOOK MANAGEMENT:

- Some recommended practices for playbook development 
are listed below:

• Use a concise description of the play's or task's purpose to name plays and tasks. The play name 
or task name is displayed when the playbook is executed. This also helps document what each 
play or task is supposed to accomplish, and possibly why it is needed. 

• Include comments to add additional inline documentation about tasks. 

• Make effective use of vertical white space. In general, organize task attributes vertically to make 
them easier to read. 

• Consistent horizontal indentation is critical. Use spaces, not tabs, to avoid indentation errors. Set 
up your text editor to insert spaces when you press the Tab key to make this easier. 

• Try to keep the playbook as simple as possible. Only use the features that you need.
  
  
