# Ansible for mineOps

After going through the [Minecraft](../Minecraft) guide for setting up and installing a Minecraft Server you probably realize a lot of effort goes into deploying a single Minecraft Server instance. This document guide will show going over all of the items in the manual guide and automating it with Ansible.

## Initial Setup

For Ansible to be able to connect to your Minecraft Server you will need to ensure Ansible can `ssh` to it. If you are just following along on your local machine, the Ansible `tasks` can have this added to the end of them to target your local machine: `delegate_to: localhost`. To have Ansible be able to `ssh` to your servers, you just need to ensure `ssh keys` are on the remote server. A guide on how to do this can be found [here](https://www.cyberciti.biz/faq/how-to-set-up-ssh-keys-on-linux-unix/).

There are 2 paths which will be outlined for using Ansible. A "basic" path which uses no features of Ansible outside of tasks in a single playbook and a "modular" path which attempts to maximize the features used (primarily for later extensibility). The basic path will be shown first.

## Deploying Minecraft Server - Basic

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

![asciicast](./mineops-ansible-basic.svg)

## Configuring Minecraft Server - Basic

TODO
- Update settings in `server.properties` (lineinfile)
- Setting custom vars for each server

## Operating a Minecraft Server

TODO
- backups
- restores
- restarts
