Ansible Roles:
----------------

**Overview**

- In the real world, play might be long and complex, with many included or imported files, and with tasks and handlers to manage various situations.
- Copying all that code into another playbook might be nontrivial work. 
- Ansible `roles` provide a way for you to make it easier to reuse Ansible code generically.
- You can package, in a standardized directory structure, all the tasks, variables, files, templates, and other resources needed to provision infrastructure or deploy applications.
- Copy that `role` from project to project simply by copying the directory. You can then simply call that role from a play to execute it. 
- A well-written `role` will allow you to pass `variables` to the `role` from the playbook that adjust its behavior, setting all the site-specific hostnames, IP addresses, user names, secrets, or other 
locally-specific details you need.
- For example, a `role` to deploy a database server might have been written to support `variables` which set the `hostname`, database admin `user` and `password`, and other parameters that need customization for your installation.
- The author of the `role` can also ensure that reasonable default values are set for those variables if you choose not to set them in the play.

**What Are Ansible Roles?**
- `Roles` are a structured way to organize playbooks into reusable components.
- Instead of writing all tasks, handlers, variables, and templates in one big playbook, you split them into roles.
- Each `role` has a standard directory layout so Ansible knows where to find things.

  Standard Role Subdirectories:
  ----------------------------

| Subdirectory | Purpose |
|--------------|---------|
| **tasks/**   | Contains the main list of tasks to execute. Usually starts with `main.yml`. |
| **handlers/**| Defines handlers (special tasks triggered by `notify`, e.g., restart a service). |
| **files/**   | Stores static files that can be copied to managed hosts. |
| **templates/**| Holds Jinja2 templates (`.j2`) for dynamic configuration files. |
| **vars/**    | Contains variables with **high precedence** (overrides defaults). |
| **defaults/**| Contains **default variables** (lowest precedence, can be overridden easily). |
| **meta/**    | Defines role metadata, including dependencies on other roles. |
| **tests/**   | Provides test playbooks and inventory for validating the role. |
| **library/** | Custom Ansible modules specific to the role. |
| **module_utils/** | Helper code for custom modules. |
| **lookup_plugins/** | Custom lookup plugins for the role. |


**Ansible roles have the following benefits:**

- `Roles` group content, allowing easy sharing of code with others. 
- `Roles` can be written that define the essential elements of a system type: web server, database server, Git repository, or other purpose 
- `Roles` make larger projects more manageable .
- `Roles` can be developed in parallel by different administrators.
  
*In addition to writing, using, reusing, and sharing your own roles, you can get roles from other sources.*
*Some roles are included as part of Red Hat Enterprise Linux, in the rhel-system-roles package.* 
*You can also get numerous community-supported roles from the Ansible Galaxy website.*
