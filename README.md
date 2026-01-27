# Ansible_automation
ansible automation
Ansible Installation steps on Ubuntu Instance

https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-ansible-on-ubuntu-20-04


#### Ansible windows setup ref:


**Link**:  https://www.devopsschool.com/tutorial/ansible/ansible-windows-adhoc-commands.html#Program1

# Ansible Master → Windows EC2 Configuration Guide

This document explains how to configure **Ansible on Ubuntu** to manage a **Windows EC2 instance** using **WinRM over HTTPS (port 5986)**.

---

## 🏗️ Architecture

| Component | OS | Role |
|-----------|----|------|
| Ansible Master | Ubuntu EC2 | Control Node |
| Target Server | Windows EC2 | Managed Node |
| Protocol | WinRM HTTPS | Communication |
| Port | 5986 | Secure WinRM |

---

# 🚀 PART 1 — Ubuntu (Ansible Master) Setup

### 1. Install Ansible
```bash
sudo apt update
sudo apt install ansible -y


2. Install WinRM dependencies
sudo apt install python3-pip -y
pip3 install pywinrm

3. Verify installation
python3 -c "import winrm"

🚀 PART 2 — Windows EC2 Setup

Login via RDP and open PowerShell as Administrator.

1. Enable WinRM
winrm quickconfig -q

2. Remove old listeners
winrm delete winrm/config/Listener?Address=*+Transport=HTTP  2>$null
winrm delete winrm/config/Listener?Address=*+Transport=HTTPS 2>$null

3. Create self-signed certificate
$cert = New-SelfSignedCertificate -CertstoreLocation Cert:\LocalMachine\My -DnsName $env:COMPUTERNAME

4. Create HTTPS listener
winrm create winrm/config/Listener?Address=*+Transport=HTTPS "@{Hostname=`"$env:COMPUTERNAME`";CertificateThumbprint=`"$($cert.Thumbprint)`"}"

5. Enable authentication
winrm set winrm/config/service/auth "@{Basic=`"true`"}"
winrm set winrm/config/service "@{AllowUnencrypted=`"true`"}"

6. Open firewall port
netsh advfirewall firewall add rule name="WinRM HTTPS 5986" dir=in action=allow protocol=TCP localport=5986

7. Verify
Get-Service WinRM
netstat -ano | findstr :5986
winrm enumerate winrm/config/listener

🚀 PART 3 — AWS Security Group
Rule	Port	Source
Custom TCP	5986	Ubuntu Security Group
🚀 PART 4 — Ansible Inventory

Edit inventory file:

sudo nano /etc/ansible/hosts

[windows]
win1 ansible_host=WINDOWS_PRIVATE_IP

[windows:vars]
ansible_user=Administrator
ansible_password=WINDOWS_PASSWORD
ansible_connection=winrm
ansible_winrm_transport=basic
ansible_port=5986
ansible_winrm_server_cert_validation=ignore
ansible_winrm_read_timeout_sec=120
ansible_winrm_operation_timeout_sec=60

🚀 PART 5 — Connectivity Testing
nc -vz WINDOWS_PRIVATE_IP 5986
curl -k https://WINDOWS_PRIVATE_IP:5986/wsman

🚀 PART 6 — Test Ansible
ansible windows -m win_ping -vvvv


Expected output:

"ping": "pong"

🧪 Test Commands
ansible windows -m win_command -a "hostname"
ansible windows -m win_shell -a "ipconfig"

📁 Sample Playbook
- name: Windows Automation Test
  hosts: windows
  tasks:
    - name: Check hostname
      win_command: hostname

    - name: Install IIS
      win_feature:
        name: Web-Server
        state: present


Run:

ansible-playbook test.yml

⚠️ Troubleshooting
Issue	Check
Timeout	Security Group port 5986
UNREACHABLE	WinRM service
Auth error	Password
Port closed	netstat on Windows

*************************************************

$ ansible win -i inventory -m win_ping 
$ ansible win -i inventory -m setup
$ ansible win -i inventory -m raw -a "dir"
$ ansible win -i inventory -m win_service -a "name=spooler"
$ ansible win -i inventory -m win_service -a "name=spooler state=stopped"
$ ansible win -i inventory -m win_service -a "name=spooler"
$ ansible win -i inventory -m win_feature -a "name=Telnet-Client state=present"    
