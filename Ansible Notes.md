# Ansible Notes & Automation Guide

## 📌 Overview
Ansible is a Configuration Management and Automation Tool used to manage multiple servers together.

### Key Features
- Free and Open Source
- Agentless Architecture
- Platform Independent
- Uses YAML-based Playbooks
- Automates:
  - Server Creation
  - Configuration Management
  - Package Installation
  - Application Deployment

---

# 🏗️ Ansible Architecture

- **Playbook** → File containing automation code
- **Inventory** → File containing server IPs
- **SSH** → Used for connecting worker nodes

> Ansible is agentless, so no software installation is needed on worker nodes.

---

# 📖 History

- Developed by **Michael DeHaan** in 2012
- Later acquired by **Red Hat**

---

# ⚙️ Setup Environment

## Create Servers
- 1 Ansible Server
- 2 Dev Servers
- 2 Test Servers

## Set Hostnames

```bash
sudo -i

hostnamectl set-hostname ansible
hostnamectl set-hostname dev-1
hostnamectl set-hostname dev-2
hostnamectl set-hostname test-1
hostnamectl set-hostname test-2
