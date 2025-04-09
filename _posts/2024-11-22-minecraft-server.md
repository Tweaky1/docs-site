---
layout: post
title: Minecraft Linux Server
date: 2024-11-22 18:21:00 +-0000
image: /assets/img/preview/minecraft-preview.png
description: Short tutorial to get a Minecraft server up and running on Linux and a reminder to myself of how my current Minecraft server installation is setup.
categories: [Guide,Reminders]
tags: [systemd,bash,games]
---

## Installing Minecraft and Dependencies

Minecraft java edition runs on, well, java. In order to get the server up and running ensure that you have the Java Runtime Environment installed. You may not be able to install the JRE on its own, just depends on your distrobution. If that is the case the JRE comes with any installation of the Development Kit.

> Examples are assuming you have a Debian based Linux distro using the `apt` package manager. Please use your provided distrobution's package manager and its syntax.
{: .prompt-info }

Use the command below to check if __openjdk__ is installed. If not use the second command to install it. At the time of writing Minecraft uses __openjdk__ version 21:

```bash
sudo apt list --installed | grep openjdk

sudo apt install openjdk-21-jdk-headless -y
```

If you are using a server OS like myself you will want the headless version meaning it will __NOT__ come with a GUI. I have never tried nor will I go over how to use the version of the JDK that comes with a built-in UI.

### Vanilla Server v.s. Alternatives

Once you have java installed it is time to decide what version of the Minecraft server you want to use. There are many different options nowadays. Reguardless of what you choose the initial setup process is the same for them all. Some will give more features than others and all of the different versions made by the community improve performance of the server. Reguardless of whether you plan to use the extended features offered by the community, I recomend you pick one of them purely for the performance improvements. Although, if you only plan to have a few players on at a time it is not necessary. If you are interested I have a list of many of the more popular versions below:

* [Bukit](https://dev.bukkit.org/)

* [Spigot](https://www.spigotmc.org/)

* [Paper](https://papermc.io/)

* [Purpur](https://purpurmc.org/)

## Installation and systemd

After making your decision on what version of the Minecraft server to use download the `.jar` file. For simplicity later on rename the file something easy to remember like `server.jar`. 

You need a folder on the server for your game files to reside. For example I have mine stored under `~/minecrat/server/`. Once you have chosen a folder move/upload the `.jar` file to that location on your server. From here we need to write and run a script that will initialize the server. Copy the below text into a new text file, name it what you like, and save it with the `.sh` extension.

> Note that the -Xmx and -Xms flags will tell java the maximum and minimum amount of ram it can use while running the serer. You can see the recomended systems requirements [here](https://minecraft.fandom.com/wiki/Server/Requirements/Dedicated).
{: .prompt-warning }

```bash
/usr/bin/java -Xmx8192M -Xms1024M -jar server.jar nogui
```
{: file="~/minecraft/server/start-server.sh" }

Once the file is created make it executable using `chmod` and run the file with `./start-server.sh`. Even if everything was done correctly, it won't get very far and will immediately shutdown. During the first launch it will create a `eula.txt` file in the same directory as your `server.jar` and start script. Set the value within to `eula=true` to accept the End User License Agreement and your server will fully function when started up from here on out.

To start the server using systemd you will need a unit file. Below is my configuration. Place the file in `/etc/systemd/system/`. Name the file whatever you like so long as it has the `.service` extension i.e. `minecraft.service`.

```
[Unit]
Description=Minecraft Server
After=network.target

[Service]
Restart=on-failure
RestartSec=10
User=USER
WorkingDirectory=/PATH/TO/SERVER/FILES/
ExecStart=/usr/bin/java -Xmx8192M -Xms1024M -jar server.jar nogui

[Install]

WantedBy=multi-user.target
```
{: file="/etc/systemd/system/minecraft.service" }

Replace "_USER_" with the user you want the service to run under, and change the "_WorkingDirectory_" path to that of your server installation. Then assuming you named the file `minecraft.service` use `sudo systemctl enable minecraft` to enable systemd to automatically start the service on boot. Use `sudo systemctl start minecraft` to start up the service for the very first time, from here on out you won't have to touch it unless updateing or configuring game files. All basic server configuration is done within the `server.properties` file. If you chose a community version of the Minecraft server, they have their own specific configuration you can play around with as well. For example __Spigot__ has the `spigot.yml` file. To shutdown the server for a version update or if you wish to edit the config files us `sudo systemctl stop minecraft` make the necessary changes and start the server again with `sudo systemctl start minecraft`.

You are just about done and ready to play the final touches will be:

* Open port `25565` on your systems firewall (if the firewall is enabled)
* Port forward the same port through your gateway to your host machine (if hosted locally)
* Give the public Ip assigned to your machine (if hosted in the cloud) or the public Ip of your home networks gateway (if hosted locally) to players
* Enter that Ip address into the correct field within Minecraft and you are all set! 

## References

[Official Minecraft Server](https://www.minecraft.net/en-us/download/server)

[Minecraft Wiki Tutorial](https://minecraft.wiki/w/Tutorials/Setting_up_a_server)
