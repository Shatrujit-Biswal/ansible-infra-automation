# 🧰 Ansible Infrastructure Automation

This project automates the provisioning and configuration of a basic **3-tier environment** using **Ansible**.  
It sets up:
- 🗄️ **Database Server (MariaDB)**
- 🌐 **Web Server (Apache/HTTPD)**
- ⚙️ **Post-install configurations** (NTP, utilities, banners)

---

## 📂 Project Structure

```
├── ansible.cfg
├── client.pem
├── inventory
├── provisioning.yaml               # Main Ansible playbook
├── roles/
│   ├── database/
│   │   ├── tasks/main.yml          # Installs & configures MariaDB
│   │   └── vars/main.yml           # Contains DB name, user, password
│   │
│   ├── post-install/
│   │   ├── defaults/main.yml       # Default NTP servers, directories
│   │   ├── handlers/main.yml       # Handlers for NTP restart
│   │   ├── tasks/main.yml          # NTP setup, utilities, banner
│   │   └── templates/
│   │       ├── ntpconf_redhat.j2
│   │       └── ntpconf_ubuntu.j2
│   │
│   └── webserver/
│       ├── defaults/main.yml       # Variables for Apache/Httpd
│       ├── tasks/main.yml          # Installs & deploys web content
│       └── templates/index.html.j2 # Sample homepage
└── .gitignore                      # Ignores .pem file
```

---

## ⚙️ Prerequisites

Before running this playbook, ensure the following:

- 🧩 **Ansible** is installed on your control node:
  ```bash
  sudo apt install ansible -y
  ```

- 🔑 You have valid SSH access to all target servers.

- 📝 Update the `inventory` file with host IPs and correct SSH users.

Example:
```yaml
all:
  hosts:
    web01:
      ansible_host: 172.31.14.60
    web02:
      ansible_host: 172.31.13.146
      ansible_user: ec2-user
    db01:
      ansible_host: 172.31.4.247

  children:
    webservers:
      hosts:
        web01:
        web02:
    dbservers:
      hosts:
        db01:
    dc_all:
      children:
        webservers:
        dbservers:
      vars:
        ansible_user: ubuntu
        ansible_ssh_private_key_file: client.pem
```

---

## 🛠️ Steps to Execute the Playbook

### 1️⃣ Clone this Repository
```bash
git clone https://github.com/Shatrujit-Biswal/Ansible.git
cd Ansible
```

### 2️⃣ Test Connection to Hosts
```bash
ansible all -m ping
```

### 3️⃣ Run the Playbook
```bash
ansible-playbook provisioning.yaml
```

✅ **What Happens:**
- Runs `post-install` on all servers
- Sets up `webserver` (Apache/Httpd)
- Configures `database` (MariaDB)
- Creates DB, user, and applies NTP settings

---

## 🧩 Role Overview

### 🔹 post-install
- Installs `ntp`, `wget`, `git`, `zip`, `unzip`
- Configures NTP based on OS (Ubuntu/RedHat)
- Sets banner `/etc/motd`
- Restarts NTP via handlers

### 🔹 webserver
- Installs Apache (`apache2` or `httpd`)
- Enables and starts service
- Deploys HTML homepage

### 🔹 database
- Installs and starts MariaDB
- Installs PyMySQL
- Creates DB and user
 ---

## 🛡️ Security

Sensitive files like SSH keys are ignored in `.gitignore`:
```gitignore
*.pem
```

---

## ⚙️ ansible.cfg

Defines default behaviors:
```ini
[defaults]
host_key_checking=False
inventory=./inventory
forks=2
log_path=/var/log/ansible.log

[privilege_escalation]
become=True
become_method=sudo
become_ask_pass=False
```

---

## 🧪 Testing Commands

Run specific role:
```bash
ansible-playbook provisioning.yaml --tags webserver
```

Run for a specific host:
```bash
ansible-playbook provisioning.yaml -l web01
```

---

## 👨‍💻 Author

**Shatrujit**  
_Ansible Projects_  
Automating cloud infrastructure — one playbook at a time 🚀
