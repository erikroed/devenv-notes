# devenv-notes
Notes to NC Global Tech Talk

# Install script

Bash file to make it easy to install Ansible on a new computer.

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

# Create playbook

We will create a really simple playbook to get us quick up and running.
First we want to create a playbook that's prints the default Hello World.

```bash
mkdir devenv
vi install.yml
```

Insert following yml

```yml
- hosts: localhost
  tasks:
    - name: Say hello
      debug:
        msg: Hello World :)
```

Verify playbook by running:

```bash
ansible-playbook install.yml
```

This should return following output:

```bash
// some output goes here
```
