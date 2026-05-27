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
- It's a Key-Value pair.
- We can use different Modules for different purpose.
- Modules are reusable components.
- Module flag is **-m**

## yum Module

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

- Playbooks are written in YAML.
- Playbooks used to execute multiple modules.
- We can reuse the Playbook multiple times.
- In real time we use a Playbook to automate our work.
- For deployment, pkg installation, server creation,...
- Here we use Key-Value pairs.
- Key-Value can also be called as Directory.
- **YAML:** Yet Another Markup Language.
- Extension for Playbook is **.yaml** or **.yaml**
- Playbook start with **--** and end with **...** (optional). 

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
- By default Ansible execute all tasks sequentially in a Playbook.
- We can use Tags to execute a specific tasks or to skip a specific tasks.

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
- We can define these variables inside the Playbook and use for multiple times
- Once a variable is defined here it will not change until we change

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

## Dynamic Variables
- These variales will be defined outside the Playbook and these will change as per our requirements.

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

## 👤 Create Multiple Users

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
- When we have two tasks in a single Playbook if task 1 is depending upon task 2 so then we can use the concept called Handlers.
- Once task 1 is executed successfully it will notify task 2 to perform the operation.
- The name of the notify and the name of the task 2 must be same.
- Triggered only when notified.

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
**Cluster:** Group of Servers
**Homogeneous:** All servers have having same OS and Flavour.
**Heterogeneous:** All servers have different OS and Flavour.

- Used to execute this module when we have different Clusters.

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
**NAME:** ansible_nodename
**FAMILY:** ansible_os_familyl
**PKG:** ansible_pkg_mgr
**CPU:** ansible_processor_cores
**MEM:** ansible_memtotal_mb
**FREE:** ansible_memfree_mb


# 🧩 Jinja2 Templates

Used for dynamic/customized outputs using variables.

---

# 🔍 Lookups
-This module used to get data from files, db and key values.

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
- Way of executing the Playbook.

## Linear
- Executes sequentially.
- If task-1 is executed on server-1 it will wait till task-2 execution.

## Free
- Executes tasks independently on all nodes.
- If task-1 is executed on server-1 it won't wait till task-2 execution.

---

# 📁 Roles 
- Roles organize playbooks into reusable structures format.
- Main purpose of Roles is to encapsulate the data.
- We can reuse the Roles multiple times.
- Length of the Playbook is decreased.
- It contains on vars, templates, tasks,...
- In real time we use Roles for our daily activities.

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
- Ansible galaxy is a website where users can share Roles and to a command-line tool for installing, creating, and managing Roles.
- Ansible galaxy gives greater visibility to one of Ansible’s most exiting features, such as application
- installation or reusable Roles for server configuration.
- Lots of people share Roles in the Ansible Galaxy.
- Ansible Roles consist of many Playbooks, which is a way to group multiple tasks into one container to do the automation in a very effective manner with clean, directory structure.
- Repository for sharing reusable roles.

```bash
ansible-galaxy init role_name
```

---

# 🔐 Ansible Vault
- It is used to encrypt the files, Playbooks, ….
- Technique: AES256 (USED BY FACEBOOK, AWS)
- Vault will store our data very safely and securely.
- If we want access any data which is in the vault we need to give a password.
- **Note:** we can restrict the users to access the Playbook also.
- Encrypt sensitive files.

## Commands

```bash
ansible-vault create creds.txt    : to create a vault

ansible-vault edit creds.txt      : to edit a vault

ansible-vault view creds.txt      : to show the content without decrypt

ansible-vault encrypt creds.txt   : to encrypt the content

ansible-vault decrypt creds.txt   : to decrypt the content

ansible-vault rekey creds.txt     : to change password for a vault
```

---

# 🐍 PIP Module
- It’s a package manager used to install python libraries/modules.

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
- Validate playbook without making changes.
- Ansible Dry Run or Ansible Check Mode feature is to validate your Playbook before execution.
- If we execute the Playbook with dry it won’t do any changes to the worker nodes.

```bash
ansible-playbook playbook.yml --check
```

---

# ⏳ Async & Polling
- For every task in Ansible we can set time limit.
- If the task is not performed in that time limit Ansible will stop Playbook execution.
- This is called as Asynchronous and Polling.

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

- **Choice:** To pass single input at a time
- **String:** To pass multiple inputs at a time
- **Multi-line String:** To pass multiple inputs on multiple lines at a time
- **File:** To pass the file as input
- **Boolean:** To pass input either **Yes** or **No**

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
