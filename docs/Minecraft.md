# Minecraft Server

Minecraft server is run with Java. There are numerous ways to accomplish this, but the first thing that is required is to locate the `server.jar` file required to run the server. This can be downloaded free of charge from Minecraft's website [here](https://www.minecraft.net/en-us/download/server/). Once the `.jar` file has been downloaded to your machine, you can in theory run the Minecraft server. There are a few things to note if you have never done this before. The first big one is that Minecraft server requires that you accept its EULA. This is fairly easy on its own, but later on we will need a way to automate this without having to do the manual method. The other is setting defaults on what the server can use. This is done by calling flags while executing the file.

## Manual setup

To deploy a Minecraft server manually, this has been outlined many times over the years and here is a [link](https://www.fosslinux.com/18111/how-to-install-minecraft-server-on-centos.htm) to an article on how to do this. However I will outline the core steps below:

* Install Java/openjdk
* Download the Minecraft Server `.jar`
* Accept the EULA (text file)
* Run the server with `java`

What this looks like is fairly straightforwarder for almost all Linux based distributions, but there are differences with configuration of various system level items between them. In this guide we will be focusing on using a RHEL based OS (CentOS, Fedora, RHEL) for consistency. Debian/Ubuntu will work as well, but some things discussed may not carry over entirely and I will do my best to call out those differences as they come up. 

---
**NOTE**

This guide assumes you will have/be `root` on your remote system. If you are having permission issues when running some commands, append them with `sudo`.

---

### Installing Java/OpenJDK

First requirement is to ensure `java` exists on the server that is going to be running Minecraft Server.

```sh
$ yum install -y java-17-openjdk
```

This will install Java version 17 which is the newest version requirement for Minecraft Server.

### Download Minecraft Server

There are a few paths in which to download the required `server.jar`, the more familiar for most people would be to go to the link in the first paragraph of this README and download the file to their machine. This option works, but isn't exactly what we are going for. We are going to download the file needed directly. There are two methods I've outlined below with one of them needing a tool called `jq` which can be obtained [here](https://github.com/stedolan/jq/releases/download/jq-1.6/jq-linux64) with a oneliner here:

```sh
$ curl -o jq https://github.com/stedolan/jq/releases/download/jq-1.6/jq-linux64 && chmod +x jq
```

Once `jq` is obtained we can download Minecraft Server.

```sh
$ MC_URL=$(curl `curl -sL https://launchermeta.mojang.com/mc/game/version_manifest.json | jq -r '.latest.release as $release | .versions[] | select (.id == $release) | .url'` | jq -r '.downloads.server.url') && curl -o server.jar $MC_URL
```

### Accept the Minecraft EULA

This is a semi-standard EULA that does need to be agreed upon (although this is strange for server software in this manner). The link to the EULA can be found [here](https://account.mojang.com/documents/minecraft_eula). The server will not start if you do not run this next bit.

```sh
$ touch eula.txt && echo "eula=true" > eula.txt
```

### Run the Minecraft Server

Now for the time we've all been waiting for, running `java`! Jokes aside, it is time to see if all of the previous steps have worked out in our favor.

```sh
$ java -Xmx1024M -Xms1024M -jar server.jar nogui
```

The above command does a few things. First it calls `java` and passes in some parameters. `-Xms1024M` sets the initial memory size available to Java and `-Xmx2048M` sets the maximum memory available to Java. The `-jar` flag says we are going to call a `jar` file, while `nogui` just tells the Minecraft Server `.jar` to run "headless" aka `nogui`.

##TODO 
- note all the files that get created when running the server
- mkdir, useradd (server manager), systemd service

## Automating Minecraft Server Installation

We can automate in a few ways, a BASH script, Ansible, Puppet and other methods. For this guide we are going to use Ansible as it doesn't require an agent to be installed and is easy to get setup and use. We more or less have a working BASH script from the previous section as all the commands can just be compounded into a single script. Lets dig into Ansible first. In this repository there is a doc for [Ansible](./Ansible.md) and should be read to get an understanding of Ansible and getting a working environment. If you are familiar with Ansible, please proceed (although refreshers come in handy from time to time).

##TODO Automation

https://github.com/KyWa/dockerbuilds
