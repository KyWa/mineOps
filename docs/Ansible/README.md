# Ansible for mineOps

After going through the [Minecraft](../Minecraft) guide for setting up and installing a Minecraft Server you probably realize a lot of effort goes into deploying a single Minecraft Server instance. This document guide will show going over all of the items in the manual guide and automating it with Ansible.

## Initial Setup

For Ansible to be able to connect to your Minecraft Server you will need to ensure Ansible can `ssh` to it. If you are just following along on your local machine, the Ansible `tasks` can have this added to the end of them to target your local machine: `delegate_to: localhost`. To have Ansible be able to `ssh` to your servers, you just need to ensure `ssh keys` are on the remote server. A guide on how to do this can be found [here](https://www.cyberciti.biz/faq/how-to-set-up-ssh-keys-on-linux-unix/).

There are 2 paths which will be outlined for using Ansible. A "basic" path which uses no features of Ansible outside of tasks in a single playbook and a "modular" path which attempts to maximize the features used (primarily for later extensibility). The basic path will be shown first.

## Basic Ansible for Minecraft Server Management

### Deploying a Minecraft Server

There is a basic playbook which handles all the tasks that were gone over in the Manual Minecraft Server Installation in this repo.

```yaml
---
- hosts: all
  vars:
    minecraft_version: "latest"

  tasks:
    # https://docs.ansible.com/ansible/latest/collections/ansible/builtin/dnf_module.html
    - name: Install Java
      dnf:
        name: java-17-openjdk
        state: installed

    # jq could be installed anywhere, but we are only checking /usr/bin/
    # https://docs.ansible.com/ansible/latest/collections/ansible/builtin/stat_module.html
    - name: Check and Set jq path
      stat: 
        path: /usr/bin/jq
      register: jq_installed

    # https://docs.ansible.com/ansible/latest/collections/ansible/builtin/group_module.html
    - name: Create Minecraft group
      group:
        name: "minecraft"
        state: present

    # https://docs.ansible.com/ansible/latest/collections/ansible/builtin/user_module.html
    - name: Create Minecraft user
      user:
        name: "minecraft"
        group: "minecraft"
        comment: "Minecraft Server Manager"
        state: present

    # https://docs.ansible.com/ansible/latest/collections/ansible/builtin/file_module.html
    - name: Create Minecraft Server Directory
      file: 
        path: /opt/minecraft
        owner: "minecraft"
        group: "minecraft"
        state: directory
        mode: 0755

    # https://docs.ansible.com/ansible/latest/collections/ansible/builtin/shell_module.html
    - name: Install jq
      shell: curl -o /usr/bin/jq -sL https://github.com/stedolan/jq/releases/download/jq-1.6/jq-linux64 && chmod +x /usr/bin/jq
      when: jq_installed.stat.exists != true

    - name: Download specific Minecraft Server
      shell: |
        curl -o /opt/minecraft/server.jar $(curl `curl -sL https://launchermeta.mojang.com/mc/game/version_manifest.json | jq -r '.versions[] | select (.id == "{{ minecraft_version }}") | .url'` | jq -r '.downloads.server.url')
      when: minecraft_version != "latest"

    - name: Download latest Minecraft Server
      shell: |
        curl -o /opt/minecraft/server.jar -sL $(curl `curl -sL https://launchermeta.mojang.com/mc/game/version_manifest.json | jq -r '.latest.release as $release | .versions[] | select(.id == $release) | .url'` | jq -r '.downloads.server.url')
      when: minecraft_version == "latest"

    - name: Set owner of server.jar
      file:
        path: /opt/minecraft/server.jar
        owner: minecraft
        group: minecraft

    # https://docs.ansible.com/ansible/latest/collections/ansible/builtin/copy_module.html
    - name: Copy EULA
      copy:
        src: eula.txt
        owner: minecraft
        group: minecraft
        dest: /opt/minecraft/eula.txt

    - name: Copy systemd service
      copy:
        src: minecraft.service
        dest: /usr/lib/systemd/system/minecraft.service

    # https://docs.ansible.com/ansible/latest/collections/ansible/posix/firewalld_module.html
    - name: Open Minecraft Server Port on Firewall
      firewalld:
        port: 25565/tcp
        permanent: yes
        state: enabled

    - name: Open Minecraft Query Port on Firewall
      firewalld:
        port: 25565/udp
        permanent: yes
        state: enabled

    # https://docs.ansible.com/ansible/latest/collections/ansible/builtin/systemd_module.html
    - name: Start Minecraft Server
      systemd:
        name: minecraft
        enabled: true
        state: started

    # Wait up to 2 minutes for Minecraft Server to become ready
    - name: Verify server is up
      shell: tail /opt/minecraft/logs/latest.log
      register: server_status
      until: server_status.stdout.find('Done') != -1
      retries: 10
      delay: 20

  collections:
    - ansible.posix
    - community.general
```

The above playbook may seem daunting as all of the tasks are in a single playbook, but this playbook will do precisely what is expected and get you to the same place as the Manual installation method. Here is a short clip of it running:

![asciicast](./mineops-ansible-basic.gif)

### Configuring a Minecraft Server

There are a few more things we would need to do for a customer, but we can move along to the more "modular" approach to Ansible and tackle these other needs. To summarize the "basic" Ansible approach we have a playbook, an inventory file and 2 Minecraft specific files we are copying over. All of this sits in a single directory and to make any lasting changes we will have to edit this file or copy it and run it the new playbook.

If we were to update the `server.properties` file on an existing server, we would have to add in multiple tasks to our single playbook to handle stopping the server, updating the property on the remote server and starting the service back up. For editing the file we can use the `lineinfile` module to update the configuration. Here is an example of the `lineinfile` module to replace the pvp parameter:

```yaml
- name: Changing the PVP parameter
  lineinefile:
    path: /opt/minecraft/server.jar
    regexp: '^pvp='
    line: "pvp=false"
```

As you can see this could get quite silly having to do this for each line if multiple parameters need to be changed. This is where moving to a more "modular" approach to Ansible makes much more sense in the long term. Let's explore the modular path in detail in the next section.

## Modular Ansible for Minecraft Server Management

### Deploying a Minecraft Server

Much like in the basic path of Ansible even if moving to a more modular approach, you will still end up with the same end result of a running Minecraft Server. One thing it does allow us to do is be able to scale easier and be able to fine tune our deployments. Lets take a look at the breakdown of the example Ansible directory in this repository.

```sh
$ tree basic_ansible/
basic_ansible/
├── eula.txt
├── hosts
├── minecraft-install.yml
└── minecraft.service

0 directories, 4 files


$ tree modular_ansible/
modular_ansible/
├── ansible.cfg
├── inventory
│   ├── group_vars
│   │   ├── customer-0
│   │   │   ├── config.yml
│   │   │   └── server-properties.yml
│   │   ├── customer-1
│   │   │   ├── config.yml
│   │   │   └── server-properties.yml
│   │   └── minecraft
│   │       ├── config.yml
│   │       └── server-properties.yml
│   ├── host_vars
│   │   ├── mineops-0.kywa.io
│   │   │   ├── config.yml
│   │   │   └── server-properites.yml
│   │   └── mineops-1.kywa.io
│   │       ├── config.yml
│   │       └── server-properites.yml
│   └── hosts
├── playbooks
│   ├── minecraft-backup.yml
│   ├── minecraft-config-change.yml
│   ├── minecraft-install.yml
│   ├── minecraft-restore.yml
│   └── minecraft-uninstall.yml
└── roles
    └── minecraft
        ├── files
        │   └── eula.txt
        ├── tasks
        │   ├── backup.yml
        │   ├── config-change.yml
        │   ├── main.yml
        │   ├── restore.yml
        │   ├── server-prep.yml
        │   ├── start-server.yml
        │   └── uninstall.yml
        └── templates
            ├── minecraft.service.j2
            └── server.properties.j2

14 directories, 27 files
```

As you can see we now have easily five times as many files as our basic Ansible setup. There is reason to this massive increase of files which we can start to dive into those reasons. If we add more customers in our basic 

### Configuring a Minecraft Server

TODO
- template `server.properties.j2`

### Operating a Minecraft Server

TODO
- backup and restore of a Minecraft Server

## Final thoughts

Hopefully throughout this doc you have seen the need if using Ansible to use a more modular approach to your directory layout.

In the next doc, we will be moving our configurations to `Git` and you can follow along [here](../Git/).
