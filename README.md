# 🧩 Ansible Overview

## 1. What is Ansible

**Ansible** is an open-source automation tool used for **configuration management**, **application deployment**, and **orchestration**.  
It works without agents, relying on **SSH** (for Linux) or **WinRM** (for Windows hosts).

Playbooks are written in **YAML** and describe desired system states in a simple, human-readable format.

---

## 2. System Requirements and Setup

### Infrastructure Components
- **Control Node** – where Ansible is installed  
- **Managed Nodes** – target servers managed by Ansible  

### Requirements
- Python 3.8 or higher  
- SSH connectivity between control and managed nodes  
- Sudo privileges on managed hosts
  ![image alt](https://github.com/yakmatic-dev/ansible-project/blob/2fcbe647c6aefd14744e85a57abc5eb3666a227e/images/ansible%20version.jpg)


### Installation Guide
Refer to the official Ansible installation documentation:  
🔗 [Ansible Installation Guide](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)

---

## 3. Configuration

The **Ansible configuration file** defines default behaviors.  
Default location: `/etc/ansible/ansible.cfg`  
A project-specific `ansible.cfg` in the working directory takes precedence.

### Example `ansible.cfg`
```ini
[defaults]
inventory = ./inventory.yml
remote_user = yakmat
host_key_checking = False
retry_files_enabled = False
log_path = /var/log/ansible.log
```

To generate a fully commented-out configuration file:
```bash
ansible-config init --disabled > ansible.cfg
```

---

## 4. Inventory Management

Inventory files define managed hosts and groups — including IPs, users, and connection parameters.

### Example `inventory.yml`
```yaml
all:
  vars:
    ansible_ssh_private_key_file: ~/.ssh/id_ansible
    ansible_python_interpreter: /usr/bin/python3

  children:
    webservers:
      vars:
        ansible_user: yakmat
      hosts:
        web01:
          ansible_host: "{{ web01_ip }}"
        web02:
          ansible_host: "{{ web02_ip }}"
          ansible_user: test

    dbservers:
      hosts:
        db1:
          ansible_host: "{{ db1_ip }}"
          ansible_user: yakmatdemo
          ansible_python_interpreter: /usr/bin/python3.6

```
#### The above ansible vault is used, creating file vault_hosts.yml and refrence it 


### Test Connection
```bash
ansible all -m ping
```

---

## 5. Ad-hoc Commands and Modules

Ad-hoc commands run quick tasks using Ansible modules.

### Example (Ping Module)
```bash
ansible all -m ping -u ubuntu --key-file ~/.ssh/id_rsa
```

See more examples in the [Ansible Module Documentation](https://docs.ansible.com/ansible/latest/collections/index_module.html).

---

## 6. Playbooks

Playbooks are YAML files that define automation steps.

### Example `site.yml`
```yaml
---
- name: Install and start Nginx
  hosts: webservers
  become: yes
  tasks:
    - name: Install nginx
      ansible.builtin.apt:
        name: nginx
        state: present

    - name: Start nginx
      ansible.builtin.service:
        name: nginx
        state: started
```

---

## 7. Variables

Variables make playbooks flexible.

```yaml
vars:
  pkg_name: nginx

tasks:
  - name: Install package
    ansible.builtin.apt:
      name: "{{ pkg_name }}"
      state: present
```

---

## 8. Debugging Variables

Display variable values using the `debug` module.

```yaml
- name: Show variable
  debug:
    var: pkg_name
```

---

## 9. Group and Host Variables

Variables can be defined per **group** or **host**.  
Fact variables (from `gather_facts`) can also be used:
```bash
ansible -m setup web01
```

### Directory Layout
```
group_vars/
  webservers.yml
host_vars/
  web01.yml
```

**Example `group_vars/webservers.yml`:**
```yaml
ansible_user: yakmat
web_package: nginx
```

---

## 10. Conditionals

Conditionals execute tasks only when specific conditions are met.

```yaml
- name: Install on Debian
  ansible.builtin.apt:
    name: nginx
  when: ansible_os_family == 'Debian'

- name: Install on RedHat
  ansible.builtin.yum:
    name: nginx
  when: ansible_os_family == 'RedHat'
```

---

## 11. Loops

Loops simplify repetitive tasks.

```yaml
- name: Install multiple packages
  ansible.builtin.apt:
    name: "{{ item }}"
    state: present
  loop:
    - git
    - curl
    - nginx
```

---

## 12. Copy and Template Modules

### Copy Module
Transfers static files.

```yaml
- name: Copy static file
  ansible.builtin.copy:
    src: files/index.html
    dest: /var/www/html/index.html
```

### Template Module
Processes Jinja2 templates for dynamic configurations.

```yaml
- name: Deploy dynamic config
  ansible.builtin.template:
    src: templates/nginx.conf.j2
    dest: /etc/nginx/nginx.conf
```

---

## 13. Handlers

Handlers run only when notified by tasks.

```yaml
tasks:
  - name: Copy config
    ansible.builtin.template:
      src: nginx.conf.j2
      dest: /etc/nginx/nginx.conf
    notify: Restart nginx

handlers:
  - name: Restart nginx
    ansible.builtin.service:
      name: nginx
      state: restarted
```

---

## 14. Roles

Roles organize playbooks into reusable components.

### Directory Structure
```
roles/
  nginx/
    tasks/main.yml
    templates/
    files/
    handlers/main.yml
    vars/main.yml
    meta/main.yml
```

### Example Role Usage
```yaml
---
- hosts: webservers
  roles:
    - nginx
```

---

## 15. Ansible Vault

Vault encrypts sensitive data such as passwords and keys.

### Commands
```bash
ansible-vault create vault.yml
ansible-vault edit vault.yml
ansible-vault view vault.yml
```

### Running Playbooks with Vault
```bash
ansible-playbook site.yml --ask-vault-pass
```

---

## 16. Ansible Galaxy

Ansible Galaxy is a repository for community-contributed roles.

```bash
ansible-galaxy search nginx
ansible-galaxy install geerlingguy.nginx
ansible-galaxy list
```

---

## 17. Troubleshooting

### Run Playbooks in Verbose Mode
```bash
ansible-playbook site.yml -vvv
```

### Common Checks
- Verify SSH connectivity  
- Confirm Python path on remote hosts  
- Review `ansible.cfg` and permissions  

---

## 18. Best Practices

- Use **roles** for structure and reuse  
- Encrypt secrets with **Ansible Vault**  
- Commit configurations to **version control**  
- Test with check mode:  
  ```bash
  ansible-playbook site.yml -C
  ```
- Keep playbooks **idempotent** (repeatable without side effects)  

---

contact : yakubiliyas12@gmail.com

 **End of Documentation**
