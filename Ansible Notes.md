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
```

## Configure SSH

```bash
passwd root

vim /etc/ssh/sshd_config
# Uncomment line 38 and 61

systemctl restart sshd
systemctl status sshd

hostname -i
```

---

# 🛠️ Install Ansible

```bash
amazon-linux-extras install ansible -y

yum install python3 python-pip python-devel -y
```

---

# 📂 Configure Inventory File

```bash
vim /etc/ansible/hosts
```

```ini
[dev]
<dev-1-private-ip>
<dev-2-private-ip>

[test]
<test-1-private-ip>
<test-2-private-ip>
```

---

# 🔑 Configure Passwordless Authentication

```bash
ssh-keygen

ssh-copy-id root@<private-ip>
```

Repeat for all worker nodes.

---

# ✅ Verify Connectivity

```bash
ansible -m ping all
```

---

# ⚡ Adhoc Commands

Used for temporary tasks.

## Examples

```bash
ansible all -a "yum install git -y"

ansible all -a "yum install maven -y"

ansible all -a "mvn --version"

ansible all -a "touch file1"

ansible all -a "yum install httpd -y"

ansible all -a "systemctl start httpd"

ansible all -a "useradd saim"

ansible all -a "cat /etc/passwd"

ansible all -a "yum remove git* maven* httpd* -y"
```

---

# 📦 Modules

Modules are reusable components.

## Yum Module

```bash
ansible all -m yum -a "name=git state=present"

ansible all -m yum -a "name=maven state=present"

ansible all -m yum -a "name=httpd state=present"
```

## Service Module

```bash
ansible all -m service -a "name=httpd state=started"

ansible all -m service -a "name=httpd state=stopped"
```

## User Module

```bash
ansible all -m user -a "name=saim state=present"

ansible all -m user -a "name=saim state=absent"
```

## Copy Module

```bash
ansible all -m copy -a "src=saim.txt dest=/tmp"
```

---

# 📜 Playbooks

Playbooks are written in YAML.

## Basic Structure

```yaml
---
- hosts: all
  tasks:

    - name: install git
      yum:
        name: git
        state: present

    - name: install httpd
      yum:
        name: httpd
        state: present

    - name: start httpd
      service:
        name: httpd
        state: started

    - name: create user
      user:
        name: jayanth
        state: present

    - name: copy file
      copy:
        src: index.html
        dest: /root
```

## Execute Playbook

```bash
ansible-playbook playbook.yml
```

---

# 🧠 Gather Facts

Used to collect worker node information.

```bash
ansible all -m setup
```

## Examples

```bash
ansible all -m setup | grep -i family

ansible all -m setup | grep -i pkg

ansible all -m setup | grep -i cores
```

---

# 🏷️ Tags

Execute specific tasks.

## Example

```yaml
- hosts: all
  tasks:

    - name: install git
      yum:
        name: git
        state: present
      tags: git

    - name: install httpd
      yum:
        name: httpd
        state: present
      tags: web
```

## Execute Single Tag

```bash
ansible-playbook playbook.yml --tags git
```

## Execute Multiple Tags

```bash
ansible-playbook playbook.yml --tags git,web
```

## Skip Tags

```bash
ansible-playbook playbook.yml --skip-tags web
```

---

# 🔄 Variables

## Static Variables

```yaml
- hosts: all

  vars:
    a: maven
    b: httpd

  tasks:

    - name: install maven
      yum:
        name: "{{ a }}"
        state: present

    - name: install httpd
      yum:
        name: "{{ b }}"
        state: present
```

---

# ⚙️ Dynamic Variables

```yaml
- hosts: all

  tasks:

    - name: install package
      yum:
        name: "{{ a }}"
        state: present
```

## Execute

```bash
ansible-playbook playbook.yml --extra-vars "a=maven"
```

---

# 🔁 Loops

Reduce repetitive code.

## Install Multiple Packages

```yaml
- hosts: all

  tasks:

    - name: install packages
      yum:
        name: "{{ item }}"
        state: present

      with_items:
        - git
        - java-1.8.0-openjdk
        - maven
        - docker
        - httpd
```

---

# 👤 Create Multiple Users

```yaml
- hosts: all

  tasks:

    - name: create users
      user:
        name: "{{ item }}"
        state: present

      with_items:
        - ravi
        - shiva
        - rajesh
        - shivani
```

---

# 🔔 Handlers

Triggered only when notified.

```yaml
- hosts: all

  tasks:

    - name: install httpd
      yum:
        name: httpd
        state: present
      notify: start httpd

  handlers:

    - name: start httpd
      service:
        name: httpd
        state: started
```

---

# 🖥️ Shell vs Command vs Raw

```yaml
- hosts: all

  tasks:

    - name: install maven
      shell: yum install maven -y

    - name: install httpd
      command: yum install httpd -y

    - name: install docker
      raw: yum install docker -y
```

> Priority:
```text
raw > command > shell
```

---

# ✅ Conditions

```yaml
- hosts: all

  tasks:

    - name: install git on RedHat
      yum:
        name: git
        state: present
      when: ansible_os_family == "RedHat"

    - name: install git on Debian
      apt:
        name: git
        state: present
      when: ansible_os_family == "Debian"
```

---

# 💡 LAMP Stack

- Linux
- Apache
- MySQL
- Python

```yaml
- hosts: all

  tasks:

    - name: install apache
      yum:
        name: httpd
        state: present

    - name: install mysql
      yum:
        name: mysql
        state: present

    - name: install python
      yum:
        name: python3
        state: present
```

---

# 🐞 Debug Module

```yaml
- hosts: all

  tasks:

    - name: print message
      debug:
        msg: "Welcome to Ansible"
```

---

# 🧩 Jinja2 Templates

Used for dynamic/customized outputs using variables.

---

# 🔍 Lookups

```yaml
- hosts: dev

  vars:
    a: "{{ lookup('file', '/root/creds.txt') }}"

  tasks:

    - debug:
        msg: "username is {{ a }}"
```

---

# 🚀 Strategies

## Linear
Executes sequentially.

## Free
Executes tasks independently on all nodes.

---

# 📁 Roles

Roles organize playbooks into reusable structures.

## Directory Structure

```text
roles/
├── pkgs/
│   └── tasks/main.yml
├── users/
│   └── tasks/main.yml
└── webserver/
    └── tasks/main.yml
```

## Master Playbook

```yaml
- hosts: all

  roles:
    - pkgs
    - users
    - webserver
```

---

# 🌌 Ansible Galaxy

Repository for sharing reusable roles.

```bash
ansible-galaxy init role_name
```

---

# 🔐 Ansible Vault

Encrypt sensitive files.

## Commands

```bash
ansible-vault create creds.txt

ansible-vault edit creds.txt

ansible-vault view creds.txt

ansible-vault encrypt creds.txt

ansible-vault decrypt creds.txt
```

---

# 🐍 PIP Module

```yaml
- hosts: all

  tasks:

    - name: install pip
      yum:
        name: python-pip
        state: present

    - name: install numpy
      pip:
        name: numpy
        state: present

    - name: install pandas
      pip:
        name: pandas
        state: present
```

---

# 🌐 Web Server vs App Server

| Server Type | Tool | Port | Path |
|---|---|---|---|
| Web Server | Apache/httpd | 80 | /var/www/html |
| App Server | Tomcat | 8080 | tomcat/webapps |

---

# 🧪 Dry Run

Validate playbook without making changes.

```bash
ansible-playbook playbook.yml --check
```

---

# ⏳ Async & Polling

```yaml
- hosts: all

  tasks:

    - name: sleep task
      command: sleep 30
      async: 10
      poll: 20
```

---

# 🔗 Jenkins Integration

## Configure Ansible Tool

```text
Manage Jenkins → Tools → Ansible
```

## Sample Pipeline

```groovy
pipeline {
    agent any

    stages {

        stage('checkout') {
            steps {
                git 'https://github.com/devopsbyraham/jenkins-java-project.git'
            }
        }

        stage('build') {
            steps {
                sh 'mvn compile'
            }
        }

        stage('test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('artifact') {
            steps {
                sh 'mvn package'
            }
        }

        stage('deploy') {
            steps {
                ansiblePlaybook(
                    installation: 'ansible',
                    inventory: '/etc/ansible/hosts',
                    playbook: '/etc/ansible/playbook.yml'
                )
            }
        }
    }
}
```

---

# ☁️ Create EC2 Using Ansible

## Install Boto

```bash
pip install boto
```

## Configure AWS CLI

```bash
aws configure
```

## EC2 Playbook

```yaml
- hosts: localhost

  tasks:

    - name: create ec2 instance
      ec2:
        region: us-east-1
        count: 3
        image: ami-04ff98ccbfa41c9ad
        instance_type: t2.micro

        instance_tags:
          Name: abc
```

---

# 🧾 Jenkins Parameters

## Types

- Choice
- String
- Multi-line String
- File
- Boolean

---

# 🔥 Advanced Jenkins Pipeline

```groovy
pipeline {
    agent any

    stages {

        stage('checkout') {
            steps {
                git branch: '$branch',
                url: 'https://github.com/devopsbyraham/jenkins-java-project.git'
            }
        }

        stage('build') {
            steps {
                sh 'mvn compile'
            }
        }

        stage('test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('artifact') {
            steps {
                sh 'mvn package'
            }
        }

        stage('deploy') {

            input {
                message "parameter check done?"
                ok "yes"
            }

            steps {
                ansiblePlaybook(
                    installation: 'ansible',
                    inventory: '/etc/ansible/hosts',
                    playbook: '/etc/ansible/deploy.yml'
                )
            }
        }
    }
}
```

---

# 📚 Important Commands Cheat Sheet

```bash
ansible -m ping all

ansible-playbook playbook.yml

ansible-playbook playbook.yml --tags tagname

ansible-playbook playbook.yml --skip-tags tagname

ansible-playbook playbook.yml --check

ansible all -m setup

ansible-vault create file.txt
```

---

# 🏁 Conclusion

Ansible is one of the most powerful DevOps automation tools used for:
- Configuration Management
- Infrastructure Provisioning
- CI/CD Automation
- Application Deployment
- Cloud Automation

It becomes extremely powerful when integrated with:
- Jenkins
- AWS
- Docker
- Kubernetes
- Terraform
