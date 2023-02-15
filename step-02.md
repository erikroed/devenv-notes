# GUI Applications

This part we will first quickly install some GUI applications like a browser and a IDE. 
In the next section, we will have a look at CLI-tools and start creating a docker image for testing our ansible script, and verify our script in isolated environments (containers) with docker.

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
- name: Create folder for installation files if it doesn't exist
  file:
    path: /opt/idea
    state: directory
    mode: '0755'
  tags:
    - ide

- name: Download IntelliJ IDEA Community Edition tarball
  get_url:
    url: https://download.jetbrains.com/idea/ideaIC-2022.3.2.tar.gz?_ga=2.151419696.80785005.1676413469-1496090286.1676413469
    dest: /opt/idea/idea.tar.gz
  tags:
    - ide

- name: Extract tarfile
  unarchive:
    src: /opt/idea/idea.tar.gz
    dest: /opt/idea
  tags:
    - ide
```

Then navigate to the binary folder:

```bash
cd /opt/idea/idea-IC-223.8617.56/bin
```

Start IntelliJ with the following command:

```bash
./idea.sh
```

This demo is done with the help of WSL, we could have extended the task to create a symlink from `/usr/local/bin/` to the binary we just executed. Or when running on your own computer, you would probably have it installed directly on your computer so it will appear with an icon in the application explorer and/or start menu.

Stop the process with `CTRL + C` (this will also stop the running program).

Now navigate back to our folder:

```bash
cd -
```

or 

```bash
cd ~/devenv
```

# CLI-Tools and Dockerfile

Go to [next page](./cli-tools.md)
