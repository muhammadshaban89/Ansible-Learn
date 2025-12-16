SELECTING HOSTS WITH HOST PATTERNS
-----------------------------------

- Host patterns are used to specify the hosts to target by a play or ad `hoc command`. 
- In its simplest form, the name of a managed host or a host group in the `inventory` is a host pattern that specifies that host or host group.
- It is usually easier to control what hosts a play targets by carefully using host patterns and having appropriate inventory groups, instead of setting complex conditionals on the play's tasks. 
- The most basic host pattern is the name for a single managed host listed in the inventory. 
- This specifies that the host will be the only one in the inventory that will be acted upon by the ansible command. 
- If an IP address is listed explicitly in the inventory, instead of a host name, then it can be used as a host pattern. 
- If the IP address is not listed in the inventory, then you cannot use it to specify the host even if the IP address resolves to that host name in the DNS.
- When a group name is used as a host pattern, it specifies that Ansible will act on the hosts that are members of the group.
-  If the host pattern is just a quoted asterisk, then `all` hosts in the inventory will match.

**Note that**

- Some characters that are used in host patterns also have meaning for the `shell`. This can be a problem when using host patterns to run `ad hoc commands` from the command line with ansible. 
- It is a recommended practice to enclose host patterns used on the command line in `single quotes` to protect them from unwanted `shell expansion`.  
- Likewise, if you are using any special wildcards or list characters in an Ansible Playbook, then you must put your host pattern in single quotes to ensure it is parsed correctly.  

```yaml
- hosts: '!test1.example.com,development'  

```
- The asterisk character can also be used to match any managed hosts or groups that contain a particular substring.
- For example, the following wildcard host pattern matches all inventory names that end in .example.com: 

```yaml
- hosts: '*.example.com' 
```

-  hosts or host groups that begin with `datacenter`.   
```yaml
- hosts: 'datacenter*' 
```
- Multiple entries in an inventory can be referenced using logical lists.
- A comma-separated list of host patterns matches all hosts that match any of those host patterns. 

```yaml
- hosts: labhost1.example.com,test2.example.com,192.168.2.2 
``` 
- If you provide a comma-separated list of groups, then all hosts in any of those groups will be targeted: 

```yaml
- hosts: dtc1,dtc2, web1,web2
```

- You can also mix managed hosts, host groups, and wildcards.

```yaml
- hosts: lab,data*,192.168.2.2 
```

**NOTE**

- The colon character (:) can be used instead of a comma. 
- However, the comma is the preferred separator, especially when working with IPv6 addresses as managed host names. 

- if an item in a list starts with an ampersand character `(&)`, then hosts must match that item in order to match the host pattern. 
- It operates similarly to a logical `AND`. 
- following host pattern matches machines in the lab group only if they are also in the `datacenter1` group: 

```yaml
- hosts: lab,&datacenter1 
```
- You could also specify that machines in the `datacenter1` group match only if they are in the `lab` group with the host patterns `&lab,datacenter1` or `datacenter1,&lab`.   
- You can exclude hosts that match a pattern from a list by using the exclamation point or `"bang"` character `(!)` in front of the host pattern. This operates like a logical `NOT`.

```yaml
- hosts: datacenter,!test2.example.com 
``` 

