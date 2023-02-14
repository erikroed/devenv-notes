# Tools

This part will evolve the playbook to install things like a web browser and a IDE, in this demo we will install IntelliJ.
We will also write tasks to install CLI tools like neovim, tmux and xpanes.

# Web browser

In this example we will install [Brave](https://brave.com/) as the web browser. Since we're configuring our own development environment,
this can be replaced or extended with similar tasks to install other browsers as well. 

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

