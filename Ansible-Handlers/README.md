ANSIBLE HANDLERS :
------------------

- Ansible modules are designed to be idempotent. This means that in a properly written playbook, the playbook and its tasks can be run multiple times without changing the managed host unless 
they need to make a change to get the managed host to the desired state. 
- However, sometimes when a task does make a change to the system, a further task may need to 
be run. For example, a change to a service's configuration file may then require that the service be 
reloaded so that the changed configuration takes effect.
- `Handlers` are tasks that respond to a notification triggered by other tasks.
- Tasks only notify their `handlers` when the task changes something on a managed host.
- Each `handler` has a globally unique name and is triggered at the end of a block of tasks in a playbook.
- If no task notifies the `handler` by name then the `handler` will not run.
- If one or more tasks notify the `handler`, the `handler` will run exactly once after all other tasks in the play have completed.
-  Because `handlers` are tasks, administrators can use the same modules in `handlers` that they would use for any other task.
-  Normally, `handlers` are used to `reboot` hosts and `restart` services.
  
- **Execution timing:** They run at the end of a play (after all tasks finish), ensuring efficiency and idempotency.
- **Purpose:** Prevent unnecessary restarts or reloads by acting only when something changes.
- **Common uses:** Restarting services (e.g., Apache, Nginx), reloading firewall rules, refreshing configuration files.

**Example:**
```yaml
- name: Deploy NGINX config with handler
  hosts: rhelnode
  become: true
  vars:
    nginx_conf_src: files/default.conf
    nginx_conf_dest: /etc/nginx/conf.d/default.conf
  tasks:
    - name: Copy NGINX default.conf to managed host
      ansible.builtin.copy:
        src: "{{ nginx_conf_src }}"
        dest: "{{ nginx_conf_dest }}"
        owner: root
        group: root
        mode: '0644'
      notify: restart nginx

  handlers:
    - name: restart nginx
      ansible.builtin.service:
        name: nginx
        state: restarted
  ```
**What does it do?**

- The copy task pushes  `default.conf` to `/etc/nginx/conf.d/`.
- If the file changes, it notifies the restart nginx handler.
- The handler runs only if needed, keeping your automation efficient.

**Some importent points about Handlers**

- 1:Handlers always run in the order specified by the `handlers` section of the play. They do not run 
in the order in which they are listed by `notify` statements in a task, or in the order in which 
tasks notify them. 
- 2:Handlers normally run after all other tasks in the play complete. A handler called by a task in the 
tasks part of the playbook will not run until all tasks under tasks have been processed. (There 
are some minor exceptions to this.) 
- 3:Handler names exist in a per-play namespace. If two `handlers` are incorrectly given the same 
name, only one will run. 
- Even if more than one task notifies a handler, the handler only runs once. If no tasks notify it, a 
handler will not run. 
- 4:If a task that includes a notify statement does not report a changed result (for example, a 
package is already installed and the task reports ok), the handler is not notified. The handler 
is skipped unless another task notifies it. Ansible notifies handlers only if the task reports the 
changed status.
- 5:Ansible treats the `notify statement` as an array and iterates over the handler names.

**EXAMPLE2:**

```yaml
- name: Deploy NGINX config and website files
  hosts: webservers
  become: true

  tasks:
    - name: Copy NGINX default.conf
      ansible.builtin.copy:
        src: files/default.conf
        dest: /etc/nginx/conf.d/default.conf
        owner: root
        group: root
        mode: '0644'
      notify:
        - reload firewall
        - restart nginx

    - name: Deploy index.html
      ansible.builtin.copy:
        src: files/index.html
        dest: /website/index.html
        owner: nginx
        group: nginx
        mode: '0644'
   
  handlers:
    - name: restart nginx
      ansible.builtin.service:
        name: nginx
        state: restarted

    - name: reload firewall
      ansible.builtin.service:
        name: firewalld
        state: reloaded
```
- Note the `notify ` section and `Handlers` section , in notify section ` - reload firewall` is notified first and then `- restart nginx
`, where as in `handlers ` section `restart nginx` is defined fisrt and then `reload firewall`.

Thanks:

👉Follow my LinkdIn Profile: www.linkedin.com/in/muhammad-shaban-45577719a
- so `handlers` section will restart `nginx` and then  reload `firewalld`. See point# 1 above.
