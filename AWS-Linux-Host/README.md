##  Steps to Configure AWS Linux Host for Ansible

### 1. Launch the EC2 Instance
- Go to AWS Console → EC2 → Launch Instance.
- Choose **Amazon Linux 2** or **Ubuntu** (both are common for Ansible labs).
- Select instance type (e.g., `t2.micro` for testing).
- Configure networking (ensure **SSH (port 22)** is open in the Security Group).
- Attach a key pair for SSH login.

---

### 2. Connect to the Instance
```bash
ssh -i ~/.ssh/mykey.pem ec2-user@<EC2_PUBLIC_IP>
```
- Default user:
  - Amazon Linux → `ec2-user`
  - Ubuntu → `ubuntu`

---

### 3. Prepare the Host for Ansible
- Update packages:
  ```bash
  sudo yum update -y   # Amazon Linux
  sudo apt update -y   # Ubuntu
  ```
- Install Python (needed for Ansible modules):
  ```bash
  sudo yum install -y python3   # Amazon Linux
  sudo apt install -y python3   # Ubuntu
  ```
- Verify:
  ```bash
  python3 --version
  ```

---

### 4. Create a Dedicated Ansible User (Optional but Recommended)
```bash
sudo adduser ansibleu
sudo passwd ansibleu
sudo usermod -aG wheel ansibleu   # Amazon Linux
sudo usermod -aG sudo ansible    # Ubuntu
```

- Configure **SSH key-based login**:
  ```bash
  sudo mkdir /home/ansibleu/.ssh
  sudo cp ~/.ssh/authorized_keys /home/ansible/.ssh/
  sudo chown -R ansible:ansible /home/ansibleu/.ssh
  sudo chmod 700 /home/ansible/.ssh
  sudo chmod 600 /home/ansible/.ssh/authorized_keys
  ```

---

### 5. Enable Passwordless Sudo
Edit sudoers:
```bash
sudo visudo
```
Add:
```
ansible ALL=(ALL) NOPASSWD:ALL
```

---

### 6. Configure Ansible Control Node
On your **Ansible control machine** (could be your laptop or another server):
- Create inventory file:
  ```ini
  [aws_hosts]
  aws1 ansible_host=<EC2_PUBLIC_IP> ansible_user=ansible ansible_ssh_private_key_file=~/.ssh/mykey.pem
  ```
- Test connectivity:
  ```bash
  ansible aws_hosts -m ping
  ```

Expected output:
```json
aws1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

**TEST**

- Create a file in current directory at yoour control node and run:
  ```bash
  ansible aws_hosts -m copy -a "src=test.txt dest=/home/ansible/"
```

---

##  Key Notes for Reliability
- Always ensure **Security Group** allows SSH from your control node.
- Use **IAM roles** if you plan to integrate AWS modules (e.g., provisioning via Ansible).
- For reproducibility, script the setup with **cloud-init** or **Ansible playbooks** so every new EC2 host is ready automatically.
