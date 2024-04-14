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
Create an inventory file that lists the Raspberry Pi nodes. You can place it in /etc/ansible/hosts or create a custom location.

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
- name: Setup Raspberry Pi nodes
  hosts: raspberry_pis
  become: yes
  tasks:
    - name: Install Python LTS
      apt:
        name: python3
        state: latest
  
    - name: Install Node Version Manager (NVM)
      shell: curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
      args:
        creates: /root/.nvm/nvm.sh
  
    - name: Install Node LTS
      shell: . ~/.nvm/nvm.sh && nvm install --lts
      args:
        executable: /bin/bash
  
    - name: Set NVM to use LTS Node as the default
      shell: . ~/.nvm/nvm.sh && nvm alias default 'lts/*'
      args:
        executable: /bin/bash
  
    - name: Install .NET LTS
      shell: wget https://packages.microsoft.com/config/ubuntu/20.04/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
        && dpkg -i packages-microsoft-prod.deb
        && apt-get update
        && apt-get install -y apt-transport-https
        && apt-get update
        && apt-get install -y dotnet-sdk-5.0
      args:
        executable: /bin/bash
    
    - name: Install AWS CLI
      shell: curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
        && unzip awscliv2.zip
        && sudo ./aws/install
      args:
        executable: /bin/bash
    
    - name: Install Git
      apt:
        name: git
        state: latest
    
    - name: Install Docker & Docker Compose
      shell: curl -fsSL https://get.docker.com -o get-docker.sh && sh get-docker.sh
        && apt-get install -y docker-compose
      args:
        executable: /bin/bash
    
    - name: Copy .bash_aliases file
      copy:
        src: /path/to/local/.bash_aliases
        dest: /home/pi/.bash_aliases
    
    - name: Configure Git credentials
      git_config:
        name: user.name
        value: Your Name
        scope: global
      git_config:
        name: user.email
        value: the.email@example.com
        scope: global
    
    - name: Copy AWS credentials
      copy:
        src: /path/to/local/.aws/credentials
        dest: /home/pi/.aws/credentials
        directory_mode: yes
```


### Step 4: Run the Playbook
After creating the playbook, run it to configure the Raspberry Pi nodes.

```bash
ansible-playbook setup.yml
```