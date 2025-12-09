Ansible Variables:
------------------
- Ansible supports variables that can be used to store values that can then be reused throughout 
files in an Ansible project.
- This can simplify the creation and maintenance of a project and reduce the number of errors. 
- Variables provide a convenient way to manage dynamic values for a given environment in your Ansible project.
- Examples of values that variables might contain include:
  
    • Users to create
  
    • Packages to install
  
    • Services to restart
  
    • Files to remove
  
    • Archives to retrieve from the internet

## Understanding Ansible :

### 1. What Are Variables?
- **Definition:** Variables store values (strings, numbers, lists, dicts) for reuse in playbooks.  
- **Purpose:** Avoid hardcoding, enable customization, and make automation reusable.  
- **Syntax:** Defined as YAML key–value pairs, referenced with `{{ var_name }}`.

**NAMING VARIABLES:**

- Variable names must start with a letter, and they can only contain letters, numbers, and 
underscores. 

|INVALID VARIABLE NAMES |  VALID VARIABLE NAMES |
|-----------------------|----------------------|
|web server             |   web_server |
|remote.file             |remote_file |
|1st file |file_1 file1 |
---

### 2. Variable Scopes
Variables can exist at different levels:

| **Scope**        | **Where Defined** | **Availability** |
|------------------|-------------------|------------------|
| **Global**       | Command line (`-e`), Ansible config | Available everywhere |
| **Play**         | Inside a playbook under `vars:` | Only within that play |
| **Host**         | In inventory or `host_vars/` | Specific to one host |
| **Group**        | In inventory or `group_vars/` | Shared by all hosts in group |
| **Role**         | Inside a role (`defaults/main.yml`, `vars/main.yml`) | Scoped to role |
| **Facts**        | Gathered automatically from hosts | Host-specific system info |

- If the same variable name is defined at more than one level, the level with the highest precedence 
wins.
- A narrow scope takes precedence over a wider scope: variables defined by the inventory are 
overridden by variables defined by the playbook, which are overridden by variables defined on the 
command line.

👉 **Precedence matters**: Command-line vars override play vars, which override group vars, etc.

---
**Defining Variables in Playbooks :**

- When writing playbooks, you can define your own variables and then invoke those values in a task. 
- For example, a variable named `web_package` can be defined with a value of httpd. A task can then 
call the variable using the `yum` module to install the `httpd` package. 
- Playbook variables can be defined in multiple ways. One common method is to place a variable in a 
`vars` block at the beginning of a playbook: -
```yaml
hosts: all 
vars: 
  user: joe 
  home: /home/joe
```
- It is also possible to define playbook variables in external files. In this case, instead of using a vars 
block in the playbook, the `vars_files` directive may be used, followed by a list of names for 
external variable files relative to the location of the playbook: -
```yaml
hosts: all 
vars_files:
  - vars/users.yml
```
The playbook variables are then defined in that file or those files in YAML format: 
```yaml
user: joe 
home: /home/joe 
```
**Using Variables in Playbooks:**

- After variables have been declared, administrators can use the variables in tasks.
- Variables are referenced by placing the variable name in double curly braces `({{ }})`.
- Ansible substitutes the variable with its value when the task is executed. 
```yaml
vars: 
  user: joe 
tasks: 
# This line will read: Creates the user joe
- name: Creates the user {{ user }}
user: 
# This line will create the user named Joe
  name:  "{{ user }}"
```
**HOST VARIABLES AND GROUP VARIABLES :**

- Inventory variables that apply directly to hosts fall into two broad categories:
  * host variables apply to a specific host, and
  * group variables apply to all hosts in a host group or in a group of host groups.
- Host variables take precedence over group variables, but variables defined by a playbook 
take precedence over both. 
- One way to define host variables and group variables is to do it directly in the inventory file. This is 
an older approach and not preferred, but you may still encounter it. 
• Defining the `ansible_user` host variable for `demo.example.com` 
```
[servers] 
demo.example.com ansible_user=joe
``` 
• Defining the user group variable for the servers host group.
```
[servers] 
demo1.example.com 
demo2.example.com 
[servers:vars] 
user=joe
```
• Defining the user group variable for the servers group, which consists of two host groups 
each with two servers. 
```
[servers1] 
demo1.example.com 
demo2.example.com 
[servers2] 
demo3.example.com 
demo4.example.com 
[servers:children]
servers1 
servers2 
[servers:vars] 
user=joe
```

*Some disadvantages of this approach are that it makes the inventory file more difficult to work 
with, it mixes information about hosts and variables in the same file, and uses an obsolete syntax*


**Using Directories to Populate Host and Group Variables :**

- The preferred approach to defining variables for hosts and host groups is to create two directories, 
`group_vars` and `host_vars`, in the same working directory as the inventory file or directory. These 
directories contain files defining group variables and host variables, respectively.

*IMPORTANT*
- The recommended practice is to define inventory variables using `host_vars` and 
`group_vars` directories, and not to define them directly in the inventory files. 
- To define group variables for the servers group, you would create a YAML file named 
`group_vars/servers`, and then the contents of that file would set variables to values using the 
same syntax as in a playbook:
```
user: joe
```
- Likewise, to define host variables for a particular host, create a file with a name matching the host 
in the `host_vars` directory to contain the host variables. 

**OVERRIDING VARIABLES FROM THE COMMAND LINE**

- Inventory variables are overridden by variables set in a playbook, but both kinds of variables may 
be overridden through arguments passed to the ansible or ansible-playbook commands on the 
command line.
- Variables set on the command line are called extra variables.
- Extra variables can be useful when you need to override the defined value for a variable for a one- 
off run of a playbook. For example: 

 ```
  #ansible-playbook main.yml -e "package=apache"

```
 

###  Best Practices
- **Organize variables** in `group_vars/` and `host_vars/` directories.  
- **Use descriptive names** to avoid conflicts.  
- **Leverage facts** (`ansible_facts`) for dynamic values.  
- **Test precedence** with `ansible-playbook --list-vars` or `debug` module.  

---

**Summary:**  
- **Host variables** → apply to one machine.  
- **Group variables** → apply to all hosts in a group.  
- **Scopes** → global, play, host, group, role, facts.  
- **Usage** → reference with `{{ var_name }}` in tasks, templates, or handlers.  


Thanks:

👉Follow my LinkdIn Profile: www.linkedin.com/in/muhammad-shaban-45577719a
