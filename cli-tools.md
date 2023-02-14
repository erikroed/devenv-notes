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
TASK [Gathering Facts] ***********************************************************************************ok: [localhost]

TASK [Update cache] **************************************************************************************ok: [localhost]

TASK [Install required system packages] ******************************************************************changed: [localhost]

TASK [Add Docker GPG apt Key] ****************************************************************************changed: [localhost]

TASK [Add Docker Repository] *****************************************************************************changed: [localhost]

TASK [Update apt and install docker-ce] ******************************************************************changed: [localhost]

TASK [Ensure group "docker" exists] **********************************************************************
ok: [localhost]

TASK [Add user to docker group] **************************************************************************
changed: [localhost]

PLAY RECAP ***********************************************************************************************
localhost                  : ok=8    changed=5    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

Now let's exit the container and add a couple of more CLI tools.

# Neovim

# Tmux

# Xpanes

