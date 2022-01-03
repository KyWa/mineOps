# Ansible Primer

This doc should be used as a primer for Ansible and using it for the purpose of automating a Minecraft Server installations and managment. If you would like a more detailed overview of Ansible, check out the documentation at https://docs.ansible.com/ansible/latest/index.html.

* Inventory File - A file containing the groups and servers to manage
* Variables - Typical key/value variables with the ability to be declared in multiple places
* Module - Contains the instructions on what do do with a specific thing (Examples: azure, files, service, package, vmware)
* Task - Outlines the specifications for a Module
* Play - Ties a task to the servers it will should be run against
* Playbook - A collection of plays
* Role - A group of specific tasks related to a "single" goal. (Example: The Minecraft role contains the tasks for installing and configuring a Minecraft Server)

## Inventory File

```ini
[groups:children] # A group of groups
group-a
group-b

[group-a] # A group called group-a
10.0.0.1  # hosts that are part of group-a

[group-a:vars] # Variables for group-a
ntp_server=ntp.example.com

[group-b] # Another group called group-b
10.0.1.1
10.0.1.2

[group-b:vars] # Variables for group-b
ntp_server=ntp.example.org
```

The above inventory file should be somewhat self explanitory, but we will outilne a few things. Groups contain servers, but can also contain groups (which will contain all servers inside all groups inside the parent group). The groups do not come into play unless a play calls the group.

## Variables

As with most applications Ansible has variables to define configuration items. Ansible allows multiple ways to declare variables with some being more logically laid out than others. Variables can be declared in the Inventory File (example above) or in a `group_vars/minecraft` directory or per server in a `host_vars/minecraft.example.com` directory. The most common format for variables is `YAML`, but even `JSON` can be used. When using Variables declared in either `group_vars` or `host_vars` all matching servers have all variable files pulled in from each of their locations. There will be more details on this later. Variables can also be declared at runtime with a `-e var_name=var_value`.

```yaml
$ cat inventory/group_vars/minecraft/config.yml
mc_port=25565
mc_motd="Welcome to mineOps!"
```

Given how Ansible can have variables declared in multiple places an order of precedence is set and can be found [here](https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html#understanding-variable-precedence).

## Modules

Ansible Modules are what grant Ansible it's power. The full Module index can be found [here](https://docs.ansible.com/ansible/2.9/modules/modules_by_category.html) and will come in quite handy as a reference when building your own Ansible playbooks.

## Task

A basic outline for a task is quite simple and follows the Ansible Module you are using. Consult the [Ansible Module Index](https://docs.ansible.com/ansible/2.9/modules/modules_by_category.html) for specific information regarding each Ansible Module. In the example below the task is creating a directory in `/opt` called `minecraft`.

```yaml
- name: Create a directory
  file:
    path: /opt/minecraft
    owner: minecraft
    group: minecraft
    mode: 0755
    state: directory
```

## Play

Plays are what Playbooks use to run Tasks or Roles against servers and groups. Building on the previous Task example we have outlned a simple Play using the same task to run against the `minecraft` group.

```yaml
---
- hosts: minecraft # This can be a server or a group
  tasks:
    - name: Create a directory
      file:
        path: /opt/minecraft
        owner: minecraft
        group: minecraft
        mode: 0755
        state: directory
```

## Playbook

A Playbook is a file which contains one or more Plays. 

## Roles

Roles are building blocks for Tasks that share a common goal. Roles are not required, but can make organizing Tasks quite simple and aid in growing your Ansible repos. Using the `minecraft` Role as an example, we can see what a Role will look like:

```sh
$ tree roles/minecraft
roles/minecraft/
├── tasks
│   ├── backup.yml
│   ├── install.yml
│   ├── main.yml
│   ├── restore.yml
│   └── uninstall.yml
└── templates
    └── server.properties.j2
```

The tasks directory contains all tasks that are part of the Role. Templates contains files that utilize the [Jinja2](https://jinja2docs.readthedocs.io/en/stable/) templating engine. The way in which these files are used can be seen in the examples [here](https://jinja.palletsprojects.com/en/3.0.x/templates/). Basic usage for a `server.properties` file would look like this with the objects in the `{{ }}` being variables declared in Ansible.

```ini
$ cat roles/minecraft/templates/server.properites.j2
pvp=true
server-port={{ mc-port }}
motd={{ mc-motd }}
hardcore=false
```


