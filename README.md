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
