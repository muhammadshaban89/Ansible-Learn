Ansible Loops:
-------------
- Using loops saves administrators from the need to write multiple tasks that use the same module. 
- For example, instead of writing five tasks to ensure five users exist, you can write one task that  iterates over a list of five users to ensure they all exist. 
- Ansible supports iterating a task over a set of items using the loop keyword.
- You can configure loops to repeat a task using each item in a list, the contents of each of the files in a list, a generated sequence of numbers, or using more complicated structures.
- A simple loop iterates a task over a list of items.
- The `loop` keyword is added to the task, and takes as a value the list of items over which the task should be iterated.
- The `loop variable item` holds the value used during each iteration.

 **EXAMPLES** 
 - This is a simple playbooks that is used to check status of services on managed hosts.
 ```bash
- name: Check Services status
  hosts: rhelnode
  tasks:
    - name: httpd is running 
      service: 
        name: httpd
        state: started
    - name: firewalld is running 
      service: 
        name: firewalld 
        state: started
```
- With `loop` playBook can be written as:
```bash
- name: Check services status 
  hosts: rhelnode
  tasks:
    - name: Checking status
      service:
        name: "{{ item }}"
        state: present
      loop:
        - httpd
        - firewalld
```

- The list used by loop can be provided by a variable. In the following example, the variable `myservices` contains the list of services that need to be running.

```bash
- name: Check services status 
  hosts: rhelnode
  vars:
    myservices:
      - httpd
      - firewalld  
  tasks:
    - name: Checking status
      service:
        name: "{{ item }}"
        state: present
      loop: "{{ myservices }}"
```

**Loops over a List of Hashes or Dictionaries:**

- The `loop` list does not need to be a list of simple values.
- Each item in the list can be actually a `hash` or a `dictionary`.
  
**EXAMPLE:**
```bash
- name: Install and enable services
  hosts: rhelnode
  tasks:
    - name: Add multiple packages with versions
      yum:
        name: "{{ item.name }}"
        state: "{{ item.state }}"
      loop:
        - { name: "httpd", state: "latest" }
        - { name: "vim", state: "present" }
  ```
  - The value of each key in the current item loop variable can be retrieved with the `item.name` and `item.state` variables, respectively.

**Earlier-Style**

- Before `Ansible 2.5`, most playbooks used a different syntax for loops.
- Multiple loop keywords were provided, which were prefixed `with with_`, followed by the name of an Ansible look-up plug-in.

**Earlier-Style Ansible Loops**

**with_items** 
- Behaves the same as the loop keyword for simple lists, such as a list of strings or a list of hashes/dictionaries. 
- Unlike `loop`, if lists of lists are provided to `with_items`, they are flattened into a single- level list. 
- The loop variable item holds the list item used during each iteration. 

**with_file**

- This keyword requires a list of control node file names.
- The loop variable item holds the content of a corresponding file from the file list during each iteration. 

**with_sequence**

- Instead of requiring a list, this keyword requires parameters to generate a list of values based on a numeric sequence.
- The loop variable item holds the value of one of the generated items in the generated sequence during each iteration.

```bash
 - name: Create Users Using Earlier-Style Ansible Loops
 - hosts: all
  become: yes
  vars:
    users_list:
      - alice
      - bob
      - charlie
      - dave
      - eve
  tasks:
    - name: Ensure users exist
      user:
        name: "{{ item }}"
        state: present
      with_items: "{{ users_list }}" 
```
