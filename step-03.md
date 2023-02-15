# CLI Tools

From now on we will focus on CLI tools and creating a Dockerfile so we can spin up containers for different environments.
This will be the more customizable ide as mentioned in the presentation.

# Dockerfile

In the part we will utilize docker, so first of we will create a new Dockerfile with the following content:

```Dockerfile
FROM ubuntu:22.04 AS base

WORKDIR /usr/local/bin

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && \
    apt-get upgrade -y && \
    apt-get install -y software-properties-common curl git build-essential && \
    apt-add-repository -y ppa:ansible/ansible && \
    apt-get update && \
    apt-get install -y curl git ansible build-essential && \
    apt-get clean autoclean && \
    apt-get autoremove --yes

ARG TAGS

COPY . .

CMD ["sh", "-c", "ansible-playbook $TAGS local.yml"]
```

The `DEBIAN_FRONTEND` env is set so the baseimage (Ubuntu:22.04) can be build without the need of setting the TimeZone.

Furthermore, when building containers with this image, we will get the necessary tools like git and ansible.
The build will also copy all of our files into the container, so we can jump straight into the container and run the
playbook without no need for more setup. Instead of re-building everytime there is a change, we can use volume mounting in docker instead, but I don't have any example on this for this demo. 

So let's give it a try. To make this process a bit easier for ourselves we will create 
one more helper script `docker-build.sh`

```bash
#!/usr/bin/bash

docker build . -t new-computer
```

Ensure this file is executable:

```bash
chmod +x docker-build.sh
```

Now we're ready to test our playbook inside a container.
First build the image to create a new container with the latest changes:

```bash
./docker-build.sh
```

Run the container:

```bash
docker run --rm -it new-computer sh
```

When inside the container, run the playbook with the docker tag:

```bash
ansible-playbook install.yml -t docker
```

This should return something like this (inside the container):

```bash
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost
does not match 'all'

PLAY [localhost] *****************************************************************************************
TASK [Gathering Facts] ***********************************************************************************
ok: [localhost]

TASK [Update cache] **************************************************************************************
ok: [localhost]

TASK [Install required system packages] ******************************************************************
changed: [localhost]

TASK [Add Docker GPG apt Key] ****************************************************************************
changed: [localhost]

TASK [Add Docker Repository] *****************************************************************************
changed: [localhost]

TASK [Update apt and install docker-ce] ******************************************************************
changed: [localhost]

TASK [Ensure group "docker" exists] **********************************************************************
ok: [localhost]

TASK [Add user to docker group] **************************************************************************
changed: [localhost]

PLAY RECAP ***********************************************************************************************
localhost                  : ok=8    changed=5    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

Now we have gotten an brief introduction of how we can test and verify the playbook in an isolated docker container. So let's add a couple of more cli tools.

# Other tools

## Neovim

Since I'm a vim user, i personally prefer to use Neovim. So here we will add a task to install this

```yml
- name: Install neovim
  become: true
  apt:
    name: neovim
    state: present
    update_cache: yes
  tags:
    - vim
```

## Tmux

When operating in the terminal it could be nice to split the terminal with windows and panes, so let's add tmux to the playbook as well:

```yml
- name: Install tmux
  apt:
    name: tmux
  tags:
    - tmux
```

## Xpanes

In co-operation with tmux we will install xpanes so we can do operations on multiple hosts, in example ping services from hosts to verify connection:

```yml
- name: Add xpanes repository
  apt_repository:
    repo: ppa:greymd/tmux-xpanes
  tags:
    - xpanes

- name: Install xpanes
  apt:
    name: tmux-xpanes
    update_cache: yes
  tags:
    - xpanes
```

# Verify updated playbook

No that we have extended the playbook with new tools, let's verify it works. Ofcource in it's own docker container.

Run:

```bash
./docker-build.sh
```

Jump into the container:

```bash
docker run --rm -it new-computer sh
```

First off, this is a new container so the only thing installed is Ansible. Let's use Ansible to install all the new tools:

```bash
ansible-playbook install.yml
```

This time, we don't use any tags. This means the playbook will run all the tasks from top to bottom, so everything we have added should be installed.

Let's verify this by running some commands from the tools we installed.

### Tmux

Let's just start a tmux session

```bash
tmux
```

### Docker

```bash
docker ps
```

### Neovim

```bash
nvim test.txt
```

Demonstrate by adding some text, change and navigate.

### Xpanes

Run ping example stolen from the xpanes github repo:

```bash
xpanes -c "ping {}" 192.168.0.{1..9}
```
