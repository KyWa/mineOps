# mineOps Series Stream - Episode 2

Supporting repository can be found here: https://github.com/KyWa/mineOps

The focus of this episode is as follows:

1. [Learning Ansible](#learning-ansible)
2. [Using Ansible to deploy a Minecraft Server](#using-ansible-to-deploy-a-minecraft-server)

Link to `README.md` from the supporting repository:

https://github.com/KyWa/mineOps/blob/master/docs/Ansible/README.md

## Learning Ansible

Below are the areas we will be focusing on:

* Ansible Terminology - Hosts, control node, etc...
* `Inventory File` - A file containing the groups and servers to manage
* `Variables` - Typical key/value variables with the ability to be declared in multiple places
* `Module` - Contains the instructions on what do do with a specific thing (Examples: azure, files, service, package, vmware)
* `Task` - Outlines the specifications for a Module
* `Play` - Ties a task to the servers it will should be run against
* `Playbook` - A collection of plays
* `Role` - A group of specific tasks related to a "single" goal. (Example: The Minecraft role contains the tasks for installing and configuring a Minecraft Server)

All of the items will be discussed were previously written in the [Ansible Primer](https://github.com/KyWa/mineOps/blob/master/docs/Ansible/ansible-primer.md) in the supporting repository.

## Using Ansible to deploy a Minecraft Server

Now that it is understood what Ansible is, lets see how we can use it to deploy and manage a Minecraft Server:

* Generate an inventory file
* Create a `playbook` that will do the work for us
* Update the `server.properties` file to make a configuration change
* Restart a Minecraft Server
* Delete a Minecraft Server
* Backup a Minecraft Server
