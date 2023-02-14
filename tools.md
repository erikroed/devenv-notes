# Tools

This part we will first quickly install some GUI applications like a browser and a IDE. 
Later on we will go deeper into CLI-tools and start creating a docker image for testing our ansible script, and verify our script in isolated environments (containers).

So let's get started with the GUI applications.

# Web browser

In this example we will install [Brave](https://brave.com/) as the web browser. Since we're configuring our own development environment,
this can be replaced or extended with similar tasks to install other browsers as well. 

Add the following yml code in `tasks/brave.yml`

```yml
- name: required libraries for brave
  become: true
  apt:
    name: ["curl", "apt-transport-https"]
  tags:
    - brave

- name: Brave Browser archive-keyring.gpg download
  get_url:
    url: https://brave-browser-apt-release.s3.brave.com/brave-browser-archive-keyring.gpg
    dest: /usr/share/keyrings/brave-browser-archive-keyring.gpg
  tags:
    - brave

- name: Brave Browser PPA setting
  become: true
  shell: echo "deb [signed-by=/usr/share/keyrings/brave-browser-archive-keyring.gpg arch=amd64] https://brave-browser-apt-release.s3.brave.com/ stable main" | tee /etc/apt/sources.list.d/brave-browser-release.list
  tags:
    - brave

- name: Brave Browser apt installation
  become: true
  apt:
    update_cache: yes
    name: brave-browser
  tags:
  - brave
```

Before we can run this, we need to adjust our `install.yml` playbook with the following line:

```yml
import_task: tasks/brave.yml
```

Now we can try running the playbook, but this time with the tag for brave:

```bash
ansible-playbook install.yml -t brave
```

This should return something like this:

```bash
[WARNING]: provided hosts list is empty, only localhost is available. Note that the implicit localhost
does not match 'all'

PLAY [localhost] *****************************************************************************************

TASK [Gathering Facts] ***********************************************************************************
ok: [localhost]

TASK [required libraries for brave] **********************************************************************
ok: [localhost]

TASK [Brave Browser archive-keyring.gpg download] ********************************************************
changed: [localhost]

TASK [Brave Browser PPA setting] *************************************************************************
changed: [localhost]

TASK [Brave Browser apt installation] ********************************************************************
changed: [localhost]

PLAY RECAP ***********************************************************************************************
localhost                  : ok=5    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

In the newer versions of WSL it is now supported to run GUI applications, so let's verify this works by opening the browser from the startmenu.

# IDE

For the IDE, we will write a task that installs [Jetbrains IntelliJ IDEA](https://www.jetbrains.com/idea/) - since I have a java background. Again, since we're writing our own dev environment, this could instead be a task to install Visual Studio or any other IDE.

Paste the following yml config in `tasks/ide.yml`

```yml

```
