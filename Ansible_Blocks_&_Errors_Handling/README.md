Ansible Blocks and Error Handling:
-----------------------------------

- In playbooks, `blocks` are clauses that logically group tasks, and can be used to control how tasks 
are executed. 

**EXAMPLE**

```yaml
- name: block example
  hosts: rhelnode
  tasks:
    - name: installing and configuring Yum versionlock plugin
      block:
        - name: package needed by yum
          yum:
            name: yum-plugin-versionlock
            state: present

        - name: lock version of tzdata
          lineinfile:
            dest: /etc/yum/pluginconf.d/versionlock.list
            line: tzdata-2016j-1
            state: present
      when: ansible_distribution == "RedHat"" 
```
- Blocks also allow for error handling in combination with the `rescue` and `always` statements.
-  If any task in a block fails, tasks in its `rescue` block are executed in order to recover.
-   After the tasks in the block clause run, as well as the tasks in the rescue clause if there was a failure, then tasks in the 
`always` clause run. To summarize:

  • block: Defines the main tasks to run. 
  • rescue: Defines the tasks to run if the tasks defined in the block clause fail. 
  • always: Defines the tasks that will always run independently of the success or failure of tasks defined in the block and rescue clauses. 

**EXAMPLE-2** 

```yaml
- name: block example
  hosts: all
  tasks:
    - name: installing and configuring Yum versionlock plugin
      block:
        - name: package needed by yum
          yum:
            name: yum-plugin-versionlock
            state: present

        - name: lock version of tzdata
          lineinfile:
            dest: /etc/yum/pluginconf.d/versionlock.list
            line: tzdata-2016j-1
            state: present

      when: ansible_distribution == "RedHat"
      rescue:
        - name: print error message
          debug:
            msg: "Failed to install or configure yum-plugin-versionlock"

      always:
        - name: notify completion
          debug:
            msg: "Block execution finished (success or failure)"
```
**What does this playbook do?*

*block section containsts two tasks**

- 1:`- name: package needed by yum`:
- This installs the YUM versionlock plugin.
- The plugin allows you to lock specific package versions so they don’t get upgraded or downgraded during `yum update`.
- Without it, YUM will always try to pull the latest available version, which can break compatibility in labs or production.

- 2:`- name: lock version of tzdata`:
- This writes a line into the versionlock list file.
- It tells YUM: “Do not upgrade or downgrade `tzdata` beyond `version 2016j-1`.”
- The `tzdata` package contains `timezone` data. Locking it ensures your system’s `timezone` rules stay consistent, which is critical for applications that rely on time accuracy (databases, logs, cron jobs, etc.).

**rescue**:
- prints message incase of failuere of `block section`

**always**:

- prints a message about block execution.
  

