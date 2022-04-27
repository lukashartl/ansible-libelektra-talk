---
title: "Ansible & Elektra"
subtitle: Talk for CM 2022S
author: Lukas Hartl
date: 12.04.2022 (recording), 27.04.2022 (lecture)
theme: Berlin
innertheme: circles
colortheme: dolphin
---

# Basics

## Ansible

* IT automation tool
* supports
  * Linux, Windows, macOS
  * network appliances: RouterOS, Cisco, ...
  * clouds: AWS, GCS, ...
* Open Source Project (RedHat)

## Getting Started

* `localhost` or some kind of connection (mostly via `ssh`)
* for Linux, Windows, ...: Python Runtime
* for Appliances/Clouds: see documentation

## Terminology: Task and Roles

* A task is the smallest unit in ansible

```yaml
- name: install htop    # description (output)
  package:              # module
    name: htop          # arguments
    state: present      # -"-
```

* A role is a sequence of tasks.
  * `webserver`

## Terminology: Inventory

* all the hosts you want to manage

* INI, YAML, ... supported

* ```yaml
  all:                            # groupname
    hosts:
      localhost:                  # host
        ansible_connection: local # arguments
  ```

* no inventory?
  * `[WARNING]: No inventory was parsed, only implicit localhost is available`

## Terminology: Playbook

* which roles/tasks to apply to which hosts/groups (inventory)

```yaml
---
- name: install webservers   # name of the playbook
  hosts: webservers          # see inventory
  roles:                     # list of roles to apply
    - mycompany.proxy
  tasks:                     # tasks (run after roles)
    - name: install nginx
      package:
        name: nginx
```

## Inventory with Remote Hosts

```yaml
all:
  hosts:
    # Linux
    example01.example.com:
      ansible_connection: ssh
      ansible_user: autouser
      ansible_password: admin123

    # Windows
    DESKTOP-123456.example.com:
      ansible_connection: ssh
      ansible_user: autouser
      ansible_password: admin123
      ansible_shell_type: powershell
```

## "Ping"

```sh
$ ansible -m ping -i inventory.yaml example01
example01 | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

# Common Tasks

## Installing software

* agnostic to OS (`apt`, `yum`, `dnf`, ...)

* ```yaml
  - name: install htop    # description (output)
    package:              # module
      name: htop          # arguments
      state: present      # -"-
  ```

* `state: present` is idempotent
  * if it is already installed, do nothing (idempotent)

## User management

* ```yaml
  - name: create user
    user:
      name: lukas
      uid: 12345
      groups:
        - admin
        - docker
  ```

## Shell commands and Idempotency

```yaml
- name: execute my cool script
  shell: /opt/cool.sh
```

* Is this idempotent?
* Ansible can't know.
  * unless you give it a hint

```yaml
- name: execute my cool script
  shell:
    cmd: /opt/cool.sh
    creates: /etc/cool.json
```

* If you want to run `shell` conditionally, have a look at `when:`
* If you only want them to run, based on whether another task has changed (trigger),
  use `handlers`

## Galaxy

* repository of ansible `roles` and `collections`
* Don't re-invent the wheel.
* Check out *Jeff Geerling's* work
  * `geerlingguy.(docker|nfs|php|postgresql)`, ...
  * <https://ansible.jeffgeerling.com>
* `elektra_initiative.libelektra`

## Deploying Containers

* `ansible-galaxy collection install community.docker`

* ```yaml
  - name: create nextcloud container
    community.docker.docker_container:
      name: nextcloud
      image: nextcloud:latest
      state: present
      exposed_ports:
        - 80
  ```

# Demo

## Demos

* Demo 1: installing packages
* Demo 2: Install libelektra on Debian
* Demo 3: Setting keys, mounting files

# Lecture

## Question: Modules

Why should we use modules like `package`, etc. over `shell`? Isn't `shell` more powerful (Pipes, ...)?

```yaml
- name: install htop
  shell: apt install htop
```

## Question: Shell

Any problems here? YES!

```yaml
- name: set Gateway
  shell: |
    echo "GATEWAY=10.0.0.1" >> /etc/sysconfig/network
```

## Configure network (using NetworkManager)

```yaml
- name: configure eno1
  nmcli:
    type: ethernet
    conn_name: 'Connection eno1'
    ifname: 'eno1'
    ip4: '10.0.0.8/24'
    gw4: '10.0.0.1'
    dns4: '1.0.0.1'
    state: present
  notify:
    - restart network
```

## Role: file structure

```
.
|-- handlers
|   |-- main.yml
|-- tasks
|   |-- main.yml
|-- templates/
|-- vars/
```
