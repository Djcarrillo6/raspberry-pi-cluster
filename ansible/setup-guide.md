## Ansible Playbook Setup Guide for Raspberry Pi Nodes
#### By: David Carrillo Jr.
#### Date: 04-14-2024


### Step 1: Install Ansible on the Control Machine
First, you need to install Ansible on the control machine (the computer you will use to manage the Raspberry Pi nodes). Ubuntu Server is recommended for the control machine as well.

```bash
sudo apt update
sudo apt install software-properties-common
sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt install ansible
```


### Step 2: Prepare Your Inventory File
Create an inventory file that lists the Raspberry Pi nodes. You can place it in `/etc/ansible/hosts` or create a custom location.

```bash
[raspberry_pis]
node1 ansible_host=192.168.1.237
node2 ansible_host=192.168.1.238
node3 ansible_host=192.168.1.242
node4 ansible_host=192.168.1.243

[raspberry_pis:vars]
ansible_user=davidcarrillojr
ansible_password=dacj6
```


### Step 3: Create the Ansible Playbook
Create a YAML file for the playbook (setup.yml). This playbook will include tasks to install software, copy files, and configure settings on all Raspberry Pi nodes.

```yaml
- name: Scaffold MicroK8 Node Environment
  hosts: raspberry_pis
  become: yes
  tasks:
    - name: Install Python LTS
      apt:
        name:
          - python3
          - python3-pip
        state: latest
    
    - name: Install Node Version Manager (NVM)
      shell: >
        curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
      args:
        creates: "{{ ansible_env.HOME }}/.nvm/nvm.sh"
      environment:
        PROFILE: /dev/null
    
    - name: Install Node LTS and set it as default using NVM
      shell: >
        export NVM_DIR="{{ ansible_env.HOME }}/.nvm" 
        && [ -s "$NVM_DIR/nvm.sh" ] 
        && \. "$NVM_DIR/nvm.sh" 
        && nvm install --lts 
        && nvm alias default 'lts/*'
      args:
        executable: /bin/bash
    
    - name: Install Microsoft repository signing key and repository
      shell: >
        wget https://packages.microsoft.com/config/ubuntu/20.04/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
        && dpkg -i packages-microsoft-prod.deb
        && rm packages-microsoft-prod.deb
      args:
        executable: /bin/bash
    
    - name: Install .NET LTS
      become: true
      apt:
        update_cache: yes
        name: 
          - apt-transport-https
          - dotnet-sdk-6.0  # Adjust to desired .NET version
        state: latest
    
    - name: Install AWS CLI
      apt:
        name: awscli
        state: latest
        update_cache: yes
    
    - name: Install Git
      apt:
        name: git
        state: latest
    
    - name: Include secret variables
      include_vars: secret_vars.yml
      no_log: true
    
    - name: Configure Git username and email
      git_config:
        name: "{{ item.name }}"
        value: "{{ item.value }}"
        scope: global
      loop:
        - { name: 'user.name', value: 'Djcarrillo6' }
        - { name: 'user.email', value: 'djcarrillo06@gmail.com' }
    
    - name: Set Git to use stored credentials
      git_config:
        name: credential.helper
        value: store
        scope: global
    
    - name: Add Git credentials to the credential store
      shell: |
        git credential approve
        protocol=https
        host=github.com
        username=Djcarrillo6
        password={{ git_token }}
      args:
        executable: /bin/bash
      no_log: true
    
    - name: Install Docker & Docker Compose
      shell: curl -fsSL https://get.docker.com -o get-docker.sh && sh get-docker.sh && apt-get install -y docker-compose && rm get-docker.sh
      args:
        executable: /bin/bash
    
    - name: Create and write to .bash_aliases
      copy:
        dest: "/home/pi/.bash_aliases"
        content: |
          p3='python3'
        owner: pi
        group: pi
        mode: '0644'
```


### Step 4: Run the Playbook
After creating the playbook, run it to configure the Raspberry Pi nodes.

```bash
ansible-playbook setup.yml
```