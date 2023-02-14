# devenv-notes
Notes to NC Global Tech Talk

# Create and navigate to folder

```bash
mkdir devenv && cd devenv
```

# Install script

Bash file to make it easy to install Ansible on a new computer.

Create file:
```bash
vi install-ansible.sh
```

Add installation script:
```bash
#!/usr/bin/bash

cat <<EOF
1. Linux (apt package manager)
2. Mac (brew package manager)
EOF

read -n 1 -p "Select your OS to install Ansible: " os

if [ $os = "1" ]; then
  sudo apt-add-repository -y ppa:ansible/ansible
  sudo apt update -y
  sudo apt install -y curl git software-properties-common ansible
elif [ $os = "2" ]; then
  brew update
  brew install git
  brew install ansible
else
  printf "\nDid not specify OS, abort installation of Ansible...\n"
fi
```

Make the file executable:
```bash
chmod +x install-ansible.sh
```

Now, let's install ansible on the computer and get started to automate our own dev environment.

```bash
./install-ansible.sh
```

# Create playbook

We will create a really simple playbook to get us quick up and running.
First we want to create a playbook that's prints the default Hello World.

```bash
vi install.yml
```

Insert following yml

```yml
- hosts: localhost
  tasks:
    - name: Say hello
      debug:
        msg: Hello, World :)
```

Verify playbook by running:

```bash
ansible-playbook install.yml
```

This should return following output:

```bash
[WARNING]: No inventory was parsed, only implicit localhost is available
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit
localhost does not match 'all'

PLAY [localhost] *******************************************************************************

TASK [Gathering Facts] *************************************************************************
ok: [localhost]

TASK [Say hello] *******************************************************************************
ok: [localhost] => {
    "msg": "Hello, World :)"
}

PLAY RECAP *************************************************************************************
localhost                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

# Install docker

As mentioned in the presentation, we are going to utilize docker to test and verify our configuration. So let's start by installing docker. Ofcourse with Ansible.

Let's open the install file again:

```bash
vi install.yml
```

This time we will replace the `Say Hello` with the following:

```yml
- hosts: localhost
  become: true
  pre_tasks:
    - name: Update cache
      apt:
        update_cache: true
      tags:
        - docker
  tasks:
    - import_tasks: tasks/docker.yml
```

This time we will make sure ansible have sudo priviliges on the machine we're running aginst. 
This is done with the `become: true`. 
The `pre_tasks` part will ensure that we update the cache before installing. This is the same as running `sudo apt update` manually in the terminal.

Now we need to add the steps to install docker as shown in the presentation. Let's create the folder tasks and create the docker.yml file:

```bash
mkdir tasks && cd tasks
vi docker.yml
```

Insert following yml code:

```yml
- name: Install required system packages
  apt:
    pkg:
      - apt-transport-https
      - ca-certificates
      - curl
      - software-properties-common
  tags:
    - docker

- name: Add Docker GPG apt Key
  apt_key:
    url: https://download.docker.com/linux/ubuntu/gpg
    state: present
  tags:
    - docker

- name: Add Docker Repository
  apt_repository:
    repo: deb https://download.docker.com/linux/ubuntu focal stable
    state: present
  tags:
    - docker
    
- name: Update apt and install docker-ce
  apt:
    name: docker-ce
    state: latest
    update_cache: true
  tags:
    - docker

- name: Ensure group "docker" exists
  ansible.builtin.group:
    name: docker
    state: present
  tags:
    - docker

- name: Add user to docker group
  become: yes
  user:
    name: "erikro"
    groups: docker
    append: yes
  tags:
    - docker
```

To verify and run this playbook, we first need to navigate back to location of the playbook file.

```bash
cd ..
```

Now we can verify the playbook by running:

```bash
ansible-playbook install.yml -t docker --check
```

By adding `--check` we're telling ansible to dry-run the playbook and not apply the changes, just tell us what would be the outcome on the host we would like to run this playbook on.
This will return and error:

```bash
[WARNING]: No inventory was parsed, only implicit localhost is available
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit
localhost does not match 'all'

PLAY [localhost] *******************************************************************************

TASK [Gathering Facts] *************************************************************************
fatal: [localhost]: FAILED! => {"ansible_facts": {}, "changed": false, "failed_modules": {"ansible.legacy.setup": {"failed": true, "module_stderr": "sudo: a password is required\n", "module_stdout": "", "msg": "MODULE FAILURE\nSee stdout/stderr for the exact error", "rc": 1}}, "msg": "The following modules failed to execute: ansible.legacy.setup\n"}

PLAY RECAP *************************************************************************************
localhost                  : ok=0    changed=0    unreachable=0    failed=1    skipped=0    rescued=0    ignored=0
```

The error message cleary states `sudo: a password is required`, so let's try running the same command again, but this time pass inn the flag so ansible will prompt for the become (sudo) password.

```bash
ansible-playbook install.yml -t docker --ask-become-pass --check
```
