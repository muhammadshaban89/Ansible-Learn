How to setup windows manged host?
---------------------------------

- you have a Linux Ansible control node and want to manage a Windows host.
- Since Ansible uses WinRM (not SSH) to talk to Windows, there are a few setup steps.


---

🔹 Steps to Manage a Windows Host with Ansible

1. Prepare the Windows Managed Host

1. Enable PowerShell Remoting (WinRM)
On the Windows machine (run PowerShell as Administrator):
```bash
winrm quickconfig
```

2. Allow basic authentication & unencrypted traffic (for testing only)
```
winrm set winrm/config/service/auth @{Basic="true"}
winrm set winrm/config/service @{AllowUnencrypted="true"}
```
⚠️ For production, it’s strongly recommended to use HTTPS with a certificate instead of unencrypted HTTP.


3. Check listening ports
```
netstat -an | findstr 5985

Port 5985 = HTTP

Port 5986 = HTTPS (needs certificate setup)
```


4. Create a dedicated Ansible user

Add a local/domain user (e.g., ansible)

Give it Administrator rights (or least privileges required).

```
New-LocalUser -Name "ansibleu" -Password (Read-Host -AsSecureString "Enter Password") -FullName "Ansible Automation User" -Description "User for Ansible automation"

Add-LocalGroupMember -Group "Administrators" -Member "ansibleu"
```
---

2. Install Required Packages on Control Node

On your Linux control node:

 ```bash
pip install pywinrm
```

 ```bash
 python3 -c "import winrm; print(winrm.__version__)"
  ```

- Optional but recommended: install `ansible.windows` collection.
  ```bash
  ansible-galaxy collection install ansible.windows
  ```
  Ansible’s Windows collection centralizes Windows modules and examples.

---

3. Configure Inventory for Windows

Edit your Ansible inventory file (/etc/ansible/hosts or custom):
```
[windows]
winhost ansible_host=192.168.244.56
[windows:vars]
ansible_user=ansibleu
ansible_password=yourwindowsuserpassword
ansible_connection=winrm
ansible_winrm_transport=basic
ansible_port=5985
```
For HTTPS + certificate auth, you’d use ansible_winrm_transport=credssp or kerberos instead.


---

4. Test the Connection

Run:
```
ansible windows -m win_ping
```
Expected output:
```
winhost | SUCCESS => {
    "changed": false,
    "ping": "pong"
}

```
---

5. Run a Sample Command
```
ansible windows -m win_shell -a "ipconfig"
```
