1: Configure AWS EC2 instance. (Windows Server)
Ensured that each Windows host is:
- Running and reachable via public/private IP.
- Configured with WinRM (HTTP) and a user (ansibleadmin) with admin privileges.
- Has firewall rules allowing WinRM traffic (default ports: 5985 for HTTP, 5986 for HTTPS).


2:Enable WinRM on Windows Hosts.
```bash
Use PowerShell (ISE)
winrm quickconfig -q
winrm set winrm/config/service/auth '@{Basic="true"}'
winrm set winrm/config/service '@{AllowUnencrypted="true"}'
Set-Item -Path WSMan:\localhost\Service\AllowUnencrypted -Value $true

# Create a new local user
New-LocalUser -Name "ansibleadminu" -Password (Read-Host -AsSecureString "Enter Password") -FullName "Ansible User" -Description "User for Ansible management"

# Add the user to Administrators group (optional, for elevated rights)
Add-LocalGroupMember -Group "Administrators" -Member "ansibleadminu"
```

3:Install Required Ansible Collections and python packages  on local system:

  a:)Install Required Python Package on Control Node On control node:
	
  ```bash
	    pip install "pywinrm>=0.3.0"
       sudo dnf install python3 python3-pip -y
```
(Optional) Install Kerberos support
If you plan to use Kerberos authentication (not required for basic auth):
```bash
  pip3 install pywinrm[kerberos]
```
Verify Installation
Run this to confirm:
```bash
  python3 -c "import winrm; print(winrm.__version__)"
```
You should see the version number (e.g., 0.4.3).
        
🧠 Pro Tip for RHEL
	If you're using a virtual environment for Ansible:

	python3 -m venv ansible-env
	source ansible-env/bin/activate
	pip install "pywinrm>=0.3.0"

This keeps your system Python clean and avoids conflicts

b:) Install ansible collections:
```bash
 ansible-galaxy collection install ansible.windows
 ansible-galaxy collection install community.windows
```

4: Create Inventory for AWS Windows hosts
```bash
[awswindows]
98.81.53.254
98.81.69.225
[awswindows:vars]
ansible_user=ansibleadmin
ansible_password="{{ vault_win_password }}"
ansible_connection= winrm
ansible_winrm_transport=basic
ansible_winrm_server_cert_validation=ignore  #for production use HTTPS 
ansible_port=5985
```
5: Secure Credentials with Ansible Vault.
```bash
ansible-vault create group_vars/awswindows.yml

vault_win_password: "YourSecurePassword"
```
6: Try to ping awswindows hosts.
```bash
ansible awswindows -m win_ping --ask-vault-pass
```
7:create a play-book.
```yaml
- name: Configure Windows Hosts
  hosts: awswindows
  gather_facts: no     #facts gathering Disabled. 
  tasks:
    - name: Ensure IIS is installed
      win_feature:
        name: Web-Server
        state: present

    - name: Create a directory
      win_file:
        path: C:\Tools
        state: directory
```
```bash
ansible-playbook aws.yml --ask-vault-pass
```




