---
layout: post
title: Create Above and Beyond Linux Server
date: 2025-4-12 20:08:00 +-0000
image: /assets/img/preview/caab-minecraft-preview.png
description: Short tutorial to get a Minecraft server up and running on Linux with the Create Above and Beyond Modpack.
categories: [Guide,Game_Servers]
tags: [systemd,bash,games,java]
---

## Create Above and Beyond Dependencies

Create Above and Beyond requires one of a few specific versions of java 8. The first issue I ran into was finding a pre-compiled version of those java versions. I ended up using `jdk8u312-b07` from Adoptium. [Here](https://adoptium.net/temurin/archive/?version=8) you will find Adoptium's download page for long term support versions of java 8. You will need to navigate to page 4, on the left hand side look for version __8.0.312+7__. Download the jdk binary for your architecture i.e. if your server has an x64 CPU you want the jdk binary labeled Linux x64. Upon completion you will have a file named `OpenJDK8U-jdk_x64_linux_hotspot_8u312b07.tar.gz`. Upload this to your server with your preferred method into a directory of your choosing. I placed it in `/opt/tar-files` using __WinSCP__. If you wish to follow along create the directory using `sudo mkdir /opt/tar-files` before copying.

There are a few steps required to setup this version java to function properly. I already had a newer version of java installed to run my vanilla game server. To allow both of them to run simultaneously requires some precautions to ensure one installation won't overwrite the other within the `$PATH` variable (from my understanding, I could be completely wrong). I don't know if this is the proper way to avoid conflicts with different java versions but this is the way I got it done. First extract the `.tar.gz` file to your desired directory using this `tar` command. I extracted mine to `/opt/jdk`.

```bash
# Output directory
sudo mkdir /opt/jdk

# Change directory to where your jdk.tar.gz file is
cd /opt/tar-files

# Extract files
sudo tar -xzf OpenJDK8U-jdk_x64_linux_hotspot_8u312b07.tar.gz -C /opt/jdk
```

Upon completion you will have a new directory: `/opt/jdk/jdk8u312-b07`. Now we will setup how to properly call or use the binary for java 8 when we want to. I simply added a symbolic link to my `/usr/bin` directory that points to the java binary in this new folder.

```bash
# Move to /usr/bin
cd /usr/bin

# Create link to the binary
sudo ln -s /opt/jdk/jdk8u312-b07/bin/java java8
```

This allows us to specifically call this version of java anytime we want using the `java8` command. This allow us to run the server without fear of any kind of conflicts later on.

## Server Installation

> At the time of writing Create Above and Beyond is on version 1.3. In the future you may have to change some of the file names within commands to reflect any changes that may have taken place.
{: .prompt-info }

First you will need to download the server files from the modpack's CurseForge page [here](https://www.curseforge.com/minecraft/modpacks/create-above-and-beyond/files/3567576). Once downloaded extract the `.zip` to where you can easily access it. Once extracted run the installer, `forge-1.16.5-36.2.20-installer.jar`, a menu will pop up. Select the "_Install server_" option and change the folder to the same one the installer is located within. It will warn you that there are already files within the folder, you can ignore it and continue. Click ok and the installer will begin downloading the necessary files. Once completed you should see a new `.jar` corresponding to your installer, and a `minecraft_server.jar`. Once you verified the existence of the two new `.jar` files you can delete the installer. To save a little bit of time later, create a new file in the same folder named `eula.txt`. Open the file and add the following text: `eula=true` and save it. Once all of that is done take all of those files and upload them to your server. On my server the are saved under the `~/minecraft/caabMinecraft`. To create the same directory ensure you are in your home directory with the `cd` command and then `mkdir -p minecraft/caabMinecraft`.

### systemd Service

Once the files are up on your server all that is left to get it running is to create a systemd unit file for it. You will need to create a new file in the `/etc/systemd/system` directory. Name it what you like, to follow along use `sudo nano /etc/systemd/system/caabMinecraft.service`, or any text editor you like over nano.

```bash
[Unit]
Description=Minecraft Server
After=network.target

[Service]
Restart=on-failure
RestartSec=10
User=USER
WorkingDirectory=/home/USER/minecraft/caabMinecraft
ExecStart=/usr/bin/java8 -Xmx8G -Xms8G -Dsun.rmi.dgc.server.gcInterval=2147483646 -XX:+UnlockExperimentalVMOptions -XX:G1NewSizePercent=0 -XX:G1ReservePercent=20 -XX:MaxGCPauseMillis=50 -XX:G1HeapRegionSize=32M -XX:+UseG1GC -jar forge-1.16.5-36.2.8.jar nogui

[Install]

WantedBy=multi-user.target
```
{: file="/etc/systemd/system/caabMinecraft.service" }

The `ExecStart=` key-value pair uses that `java8` command we "_created_" earlier.

> Note that the -Xmx and -Xms flags will tell java the maximum and minimum amount of ram it can use while running the serer. In this instance they are defined in gigabytes. You can see the recommended systems requirements [here](https://minecraft.fandom.com/wiki/Server/Requirements/Dedicated).
{: .prompt-warning }

Replace all instances of "_USER_" with the name for your user account that you want to run the server. If you are __not__ following exactly you will need to replace the "_WorkingDirectory_" with the absolute path to your server files. Once you have the file sorted out run these commands:

```bash
# Enable automatic restart of the server on boot and failures
sudo systemctl enable caabMinecraft

# Start the serer for the first time manually
sudo systemctl start caabMinecraft
```

Should you need to stop the serer for any reason like updates run `sudo systemctl stop caabMinecraft` apply the changes. Once completed you will need to start the server again manually with `sudo systemctl start caabMinecraft`. Similarly to a vanilla Minecraft server it will create some new files when you start it for the first time: `server.properties`, `banned-ips.json`, `banned-players.json`, `ops.json`, and `whitelist.json`. All of these files are used in configuring the server. If you have another minecraft server running on your network you will need to change both the `server-port` and the `query.port` off of the default of 25565, assuming your other server uses the default port.

You are just about done and ready to play the final touches will be:

* Open the port defined in `server.properties` on your systems firewall (if the firewall is enabled)
* Port forward the same port through your gateway to your host machine (if hosted locally)
* Give the public Ip assigned to your machine (if hosted in the cloud) or the public Ip of your home networks gateway (if hosted locally) to players
* Enter that Ip address into the correct field within your modded version of Minecraft and you are all set! 

## References

[Create Above and Beyond Wiki](https://github.com/simibubi/Above-and-Beyond/wiki)

[Create Above and Beyond CurseForge](https://www.curseforge.com/minecraft/modpacks/create-above-and-beyond)
