MANAGING TASK ERRORS IN PLAYS :
------------------------------

- Ansible evaluates the return code of each task to determine whether the task `succeeded` or `failed`. 
- Normally, when a task fails Ansible immediately aborts the rest of the play on that host, skipping all subsequent tasks. 
- But sometimes you might want to have play execution continue even if a task fails.
- For example, you might expect that a particular task could fail, and you might want to recover by running some other task conditionally.
- There are a number of Ansible features that can be used to manage task errors.
  
**1-Ignoring Task Failure:**

- By default, if a task fails, the play is aborted. However, this behavior can be overridden by ignoring failed tasks.
- You can use the `ignore_errors` keyword in a task to accomplish this.
  
**EXAMPLE:**
```yaml
- name: Install Latest Packages
  hosts: rhelnode
  gather_facts: no
  tasks:
    - name: install xyz package
      yum:
        name: xyz
        state: latest
      ignore_errors: yes

    - name: install new package
      yum:
        name: curl
        state: latest

          #first task will fail as "xyz" is not a pckg
          #so play will be aborted if ignore_errors: yes is not mentioned or commented

```

**2-Forcing Execution of Handlers after Task Failure**

- Normally when a task fails and the play aborts on that host, any handlers that had been notified by earlier tasks in the play will not run.
- If you set the `force_handlers: yes` keyword on the play, thennotified `handlers` are called even if the play aborted because a later task failed.
```yaml
- name: Test Force Handlers
  gather_facts: no
  hosts: rhelnode
  force_handlers: yes
  tasks:
    - name: Install nginx
      yum:
        name: nginx
        state: present
      notify: restart nginx

    - name: Simulate a failure
      command: /bin/false

  handlers:
    - name: restart nginx
      service:
        name: nginx
        state: restarted
```
**Best Practice**
- Use `force_handlers: yes` sparingly. It’s powerful but can mask failures if you rely on handlers to “fix” things.
- Combine with `ignore_errors` or `failed_when` for fine-grained control.

**3-Specifying Task Failure Conditions:**

- You can use the `failed_when` keyword on a task to specify which conditions indicate that the task has failed.
- This is often used with `command` modules that may successfully execute a command, but the command's output indicates a failure.
- `failed_when` is one of the most powerful Ansible directives because it lets you define custom failure conditions for a task.
- Instead of relying only on the command’s exit code, you can decide when a task should be considered failed (or not).

```yaml
 
- name: Demo of failed_when usage
  hosts: rhelnode
  gather_facts: no

  tasks:
    - name: Run a health check script
      command: /usr/local/bin/healthcheck.sh
      register: health_result
      changed_when: false
      failed_when:
        - "'ERROR' in health_result.stdout"
        - health_result.rc != 0

    - name: Show health check output
      debug:
        var: health_result.stdout
  ```
  
***Best practice:**

- Use `failed_when` for fine‑grained control instead of blindly using `ignore_errors`.
- Always combine with `registe`r to capture task results.
- Keep conditions explicit and readable for maintainability.

**4:Specifying When a Task Reports “Changed” Results**

- When a task makes a change to a managed host, it reports the `changed` state and notifies handlers.
- When a task does not need to make a change, it reports ok and does not notify handlers.
- The `changed_when` keyword can be used to control when a task reports that it has changed.
- n Ansible, the `changed_when` directive lets you control whether a task is marked as `changed` or not.
- By default, Ansible decides based on the module’s behavior (e.g., copy marks changed if the file differs).
- With `changed_when`, you override that logic ,itsuseful for idempotency and cleaner playbook run.
- `changed_when: false` keeps validation tasks from cluttering your playbook output with “changed” states.
- Conditional `changed_when` lets you highlight important state changes (like high disk usage) without actually modifying anything.
- Combining with `failed_when` gives you precise control over both change detection and failure logic.

```yaml
---
- name: Demonstrate changed_when usage
  hosts: rhelnode
  become: yes

  tasks:
    - name: Ensure nginx is installed
      yum:
        name: nginx
        state: present

    - name: Validate nginx configuration
      command: nginx -t
      register: nginx_test
      changed_when: false        # Validation should never mark as changed
      failed_when: "'successful' not in nginx_test.stderr"

    - name: Check disk usage
      shell: df -h / | awk 'NR==2 {print $5}' | sed 's/%//'
      register: disk_usage
      changed_when: disk_usage.stdout|int > 80   # Mark changed if usage > 80%
      failed_when: disk_usage.stdout|int > 90    # Fail if usage > 90%

    - name: Debug disk usage
      debug:
        msg: "Disk usage is {{ disk_usage.stdout }}%"
```
**EXAMPLE `changed_when: false`:

```yaml
---
- name: Validate NGINX service
  hosts: rhelnode
  gather_facts: no

  tasks:
    - name: Check NGINX status
      command: systemctl status nginx
      register: nginx_status
      changed_when: false
      failed_when: "'inactive' in nginx_status.stdout"

    - name: Show NGINX status output
      debug:
        var: nginx_status.stdout
```

*What this playbook do?*

- Install `nginx: Standard` package task.
- Validate nginx config:
- Runs `nginx -t`.
- Always shows as `ok` (not changed) because it’s just a check.
- Fails if “successful” isn’t in the output.
- Check disk usage:
- Marks task as changed if usage > 80%.
- Fails if usage > 90%.
- Debug output: Shows the actual disk usage.

**5-Ansible Blocks and Error Handling:**

- In playbooks, `blocks` are clauses that logically group tasks, and can be used to control how tasks 
are executed. 

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

Thanks:

👉Follow my LinkdIn Profile: www.linkedin.com/in/muhammad-shaban-45577719a
  
