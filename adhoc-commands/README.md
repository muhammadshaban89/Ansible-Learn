What are adhoc-commands in Ansible?
-----------------------------------
- An ad hoc command is a way of executing a single Ansible task quickly, one that you do not need to save to run again later.
-  They are simple, online operations that can be run without writing a playbook.
-  Use the ansible command to run ad hoc commands:
  
```
ansible host-pattern -m module [-a 'module arguments'] [-i inventory]
```
- The host-pattern argument is used to specify the managed hosts on which the ad hoc command should be run.
- It could be a specific managed host or host group in the inventory.
- The `-m` option takes as an argument the name of the module that Ansible should run on the targeted hosts.
-  The `-a` option takes a list of those arguments as a quoted string.

  **EXAMPLE**

  ```
ansible all -m ping
```

**When to Use Ad Hoc Commands:**

- Quick troubleshooting (e.g., test SSH connectivity).
- Immediate system changes (e.g., restart services).
- Gathering facts (e.g., system info with  module).
- One‑time tasks where writing a playbook is unnecessary.

**🔹 Best Practices**

- Use playbooks for repeatable, complex tasks.
- Keep ad hoc commands for short, urgent, or exploratory actions.
- Always specify inventory groups to avoid accidental changes across all hosts.
