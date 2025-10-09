
## What is Ansible

Ansible is an open-source automation tool used for configuration management, application deployment, and orchestration. It works without agents, relying on SSH for Linux or WinRM for Windows hosts. Playbooks are written in YAML and describe desired system states in a simple and human-readable format.


## 2. System Requirements and Setup
Infrastructure typically includes:
- Control Node – where Ansible is installed.
- Managed Nodes – target servers managed by Ansible.
- Requirements:
- Python 3.8 or higher
- SSH connectivity between control and managed nodes
- Sudo privileges on managed hosts

Installation Guide:
go to "https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html" 
The link above is a docx for installing ansible on diffrent enviroment

## 3. Configuration
The configuration file defines Ansible behavior. Default location: /etc/ansible/ansible.cfg. 
A project-specific ansible.cfg in the working directory takes precedence.

Example:
[defaults]
inventory = ./inventory.yml
remote_user = yakmat
host_key_checking = False
retry_files_enabled = False
log_path = /var/log/ansible.log

You can generate a fully commented-out example ansible.cfg file, for example:

$ ansible-config init --disabled > ansible.cfg

## 4. Inventory Management
Inventory files define managed hosts and groups info iike ip addresses, username, password, certs or basically the info the target(remote) machine needs


Test connection:
ansible all -m ping

## 5. The ad-hoc commands using module
Ping module verifies connectivity:
ansible all -m ping -u ubuntu --key-file ~/.ssh/id_rsa
we can check ansible docx on more ad-hoc cmds


## 7. Variables
Variables make playbooks flexible.

Example:
vars:
  pkg_name: nginx

tasks:
  - name: Install package
    ansible.builtin.apt:
      name: "{{ pkg_name }}"
      state: present
## 8. Debugging Variables
Use debug to display variable values.
Example:
**```**
- name: Show variable
  debug:
    var: pkg_name
 **```**
## 9. Group and Host Variables
Variables can be defined per group or host ... though variable takes priority and there is **fact variable** that are generated from gather_facts
$ ansible -m setup web01

Directory layout:
group_vars/
  webservers.yml
host_vars/
  web01.yml

group_vars/webservers.yml:
ansible_user: yakmat
web_package: nginx
## 10. Conditionals
Conditionals make tasks run only when certain conditions are met. "when" command is usually used

Example:
**```**
- name: Install on Debian
  ansible.builtin.apt:
    name: nginx
  when: ansible_os_family == 'Debian'
- name: Install on RedHat
  ansible.builtin.yum:
    name: nginx
  when: ansible_os_family == 'RedHat'
**```**
## 11. Loops
Loops repeat tasks for multiple items.

Example:
**```**
- name: Install multiple packages
  ansible.builtin.apt:
    name: "{{ item }}"
    state: present
  loop:
    - git
    - curl
    - nginx
**```**
## 12. Copy and Template Modules
Copy module transfers static files.
Example:
**```**
- name: Copy static file
  ansible.builtin.copy:
    src: files/index.html
    dest: /var/www/html/index.html
**```**
  
Template module processes Jinja2 templates. template is dynamic and it is going to read your file from template file 
Example:
**```**
- name: Deploy dynamic config
  ansible.builtin.template:
    src: templates/nginx.conf.j2
    dest: /etc/nginx/nginx.conf
**```**
## 13. Handlers
Handlers are triggered when notified by tasks.

Example:
yaml
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

## 14. Roles
Roles organize playbooks into reusable components.

### Structure:
roles/
  nginx/
    tasks/main.yml
    templates/
    files/
    handlers/main.yml
    vars/main.yml
    meta/main.yml

Playbook usage:
---
- hosts: webservers
  roles:
    - nginx
## 15. Ansible Vault
Vault secures sensitive data.

Create and manage:
ansible-vault create vault.yml
ansible-vault edit vault.yml
ansible-vault view vault.yml

Run playbooks with vault:
ansible-playbook site.yml --ask-vault-pass
## 16. Ansible Galaxy
Ansible Galaxy is a public repository for sharing roles.

### Search and install roles:
ansible-galaxy search nginx
ansible-galaxy install geerlingguy.nginx

### List installed roles:
ansible-galaxy list
## 17. Troubleshooting
Run playbooks in verbose mode for details:
ansible-playbook site.yml -vvv

### Common checks:
- Verify SSH connectivity
- Confirm Python path on remote hosts
- Review ansible.cfg and permissions
## 18. Best Practices
- Use roles for structure and reuse
- Encrypt secrets with vault
- Commit configurations to version control
- Test with check mode: ansible-playbook site.yml -C
- Keep playbooks idempotent
  
  







