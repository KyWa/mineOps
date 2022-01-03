# Ansible for mineOps

After going through the [Minecraft](../Minecraft) guide for setting up and installing a Minecraft Server you probably realize a lot of effort goes into deploying a single Minecraft Server instance. This document guide will show going over all of the items in the manual guide and automating it with Ansible.

## Initial Setup

For Ansible to be able to connect to your Minecraft Server you will need to ensure Ansible can `ssh` to it. If you are just following along on your local machine, the Ansible `tasks` can have this added to the end of them to target your local machine: `delegate_to: localhost`. To have Ansible be able to `ssh` to your servers, you just need to ensure `ssh keys` are on the remote server. A guide on how to do this can be found [here](https://www.cyberciti.biz/faq/how-to-set-up-ssh-keys-on-linux-unix/).

## Deploying Minecraft Server

TODO
- package install
- grab binary
- create `eula`
- mkdir for server
- create service
- start server

## Configuring Minecraft Server

TODO
- Update settings in `server.properties`
- Setting custom vars for each server

## Operating a Minecraft Server

TODO
- backups
- restores
- restarts
