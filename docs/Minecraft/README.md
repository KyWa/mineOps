# Minecraft Server

Minecraft server is run with Java. There are numerous ways to accomplish this, but the first thing that is required is to locate the `server.jar` file required to run the server. This can be downloaded free of charge from Minecraft's website [here](https://www.minecraft.net/en-us/download/server/). Once the `.jar` file has been downloaded to your machine, you can in theory run the Minecraft server. There are a few things to note if you have never done this before. The first big one is that Minecraft server requires that you accept its EULA. This is fairly easy on its own, but later on we will need a way to automate this without having to do the manual method. The other is setting defaults on what the server can use. This is done by calling flags while executing the file.

## Manual Minecraft Server Installation

To deploy a Minecraft server manually, this has been outlined many times over the years and here is a [link](https://www.fosslinux.com/18111/how-to-install-minecraft-server-on-centos.htm) to an article on how to do this. However I will outline the core steps below:

* Install Java/openjdk
* Download the Minecraft Server `.jar`
* Accept the EULA (text file)
* Run the server with `java`

What this looks like is fairly straightforwarder for almost all Linux based distributions, but there are differences with configuration of various system level items between them. In this series we will be focusing on using a RHEL based OS (CentOS, Fedora, RHEL) for consistency. Debian/Ubuntu will work as well, but some things discussed may not carry over entirely and I will do my best to call out those differences as they come up. 

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

The above command does a few things. First it calls `java` and passes in some parameters. `-Xms1024M` sets the initial memory size available to Java and `-Xmx2048M` sets the maximum memory available to Java. The `-jar` flag says we are going to call a `jar` file, while `nogui` just tells the Minecraft Server `.jar` to run "headless" aka `nogui`. If the `nogui` option is omitted, your machine will launch whichever Java application UI is available and show the same data as your console, but in a UI which is not needed for most things.

---
**NOTE**

If you try to connect you may recieve a "Connection Refused", this is most likely an indication that the Firewall on your server (or local machine) is closed. You can open the port needed for Minecraft Server with this command: `firewall-cmd --add-port=25565/tcp`. Note this isn't permanent, if you are running the setup as it is then you will run this: `firewall-cmd --add-port=25565/tcp --permanent && firewall-cmd --reload`. Please note that the rest of the `mineOps` series will not be using this method of running a server, but if all you wish to do is stand up a Minecraft Server on RHEL/CentOS/Fedora, you are good to go.

---

### Inspecting the files created by Minecraft Server

If you check the directory where you are running the server (either CTRL+C the previous `java -jar` command or open another terminal connection) you will notice a number of files as seen here:

```sh
$ ls -lF
total 45272
-rw-r--r--.  1 root root        2 Dec 27 18:43 banned-ips.json
-rw-r--r--.  1 root root        2 Dec 27 18:43 banned-players.json
-rw-r--r--.  1 root root       10 Dec 27 18:43 eula.txt # We created this
drwxr-xr-x.  8 root root       77 Dec 27 18:41 libraries/
drwxr-xr-x.  2 root root       51 Dec 27 18:43 logs/
-rw-r--r--.  1 root root        2 Dec 27 18:43 ops.json
-rw-r--r--.  1 root root 46324407 Dec 23 01:03 server.jar # This is the jar file we downloaded
-rw-r--r--.  1 root root     1065 Dec 27 18:43 server.properties
-rw-r--r--.  1 root root      109 Dec 27 18:46 usercache.json
drwxr-xr-x.  3 root root       20 Dec 27 18:41 versions/
-rw-r--r--.  1 root root        2 Dec 27 18:43 whitelist.json
drwxr-xr-x. 12 root root     4096 Dec 27 18:50 world/
```

As you can see a number of files get created, but only a few of these are useful for the purpose of this series. If you wish to know more about these files, you can visit [this Wiki](https://minecraft.fandom.com/wiki/Minecraft_Wiki) and search for the files you wish to know more about. For this series we are primarily going to focus on `server.properties` as it contains most of what we are concerned with regarding running and configuring a Minecraft Server.

For the rest of manually running a server we are basically done. There are some for sure "nice to haves" which I will outline here. Having a dedicated directory for the server to run in, a user for the server to run as, and a system service to keep the server running. Each of these things is handled quite easily with these commands:

```sh
$ mkdir -p /opt/minecraft && cp server.jar /opt/minecraft && useradd minecraft
$ cat << EOF >> minecraft.service
[Unit]
Description=start and stop the minecraft-server

[Service]
WorkingDirectory=/opt/minecraft/

User=minecraft
Group=minecraft
Restart=on-failure
RestartSec=20 5

ExecStart=/usr/bin/java -Xms512M -Xmx3048M -jar server.jar nogui

[Install]
WantedBy=multi-user.target
EOF

$ cp minecraft.servive /usr/lib/systemd/system/
$ systemctl enable --now minecraft
```

Once the above code sections are run, you will have a Minecraft Server running and will be started when the server starts. Note all of the effort it took above in this portion of the guide to get one Minecraft Server running. Some things you might do with this server after running for some time would be to change the parameters in `server.properties`. What if you are running multiple servers? You would have to connect to each server and update those parameters for each server instance you are running. This is where automation kicks in and now we will begin to focus on the automation portion of this series.

## Automating Minecraft Server Installation

After having gone through the [Manual Minecraft Server Installation[(#manual-minecraft-server-installation) you will have had a running Minecraft Server installation, but it was on a single server and has quite a few steps to get it all up and running. What if we wanted to run multiple Minecraft Servers? Would you be willing to manually login to each server and go through each of the steps above? Or what if you wanted to change parameeters on each of these servers and have to worry about keeping a list of which servers had which configuration?

We can automate in a few ways, a BASH script, Ansible, Puppet and other methods. For this series we are going to use Ansible as it doesn't require an agent to be installed and is easy to get setup and use. We more or less have a working BASH script from the previous section as all the commands can just be compounded into a single script. Lets dig into Ansible first. In this repository there is a doc for [Ansible](../Ansible) and should be read to get an understanding of Ansible and getting a working environment. If you are familiar with Ansible, please proceed to the next section (although refreshers come in handy from time to time).

### Ansible Playbook

##TODO
- playbook for install
- update configurations
- server backup
