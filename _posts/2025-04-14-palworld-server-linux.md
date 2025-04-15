---
layout: post
title: Palworld Linux Server
date: 2025-04-14 00:00:00 +-0000
image: /assets/img/preview/palworld-preview.png
description: Short tutorial to get a Palworld server up and running on Linux.
categories: [Guide,Game_Servers]
tags: [systemd,steamcmd,bash,games]
---

## Palworld and Dependencies

[__steamcmd__](https://developer.valvesoftware.com/wiki/SteamCMD) is the primary tool that we will use in the creation of our Palworld dedicated server. Double check if __steamcmd__ is installed. If not install it before proceeding.

> Installation command assumes you have a Debian based Linux distribution using the `apt` package manager. Please use your provided distribution's package manager.
{: .prompt-info }

```bash
which steamcmd

sudo apt install steamcmd -y
```

If the which command is successful and __steamcmd__ is already installed you will see the absolute path to it's corresponding binary file. My output, for example, was `/usr/games/steamcmd`. If it fails there will be no output, and you will have to install it.

Once __steamcmd__ is installed we can use the _Steam ID_ of Palworld's dedicated server to install the necessary files. Palworld's server ID is `2394010`. Run and save the below command to a file, this same command can be used at a later time to update the game. I saved mine to `~/bin/update-palworld`. If you decide to save it to a file you need to make it executable with `chmod 744 update-palworld`.

```bash
steamcmd +force_install_dir /PATH/YOU/WANT/GAME/FILES app_update 2394010 validate +exit
```
{: .file="~/bin/update-palworld" }

Once Palworld is installed all the important assets and configuration files required for the game to function are in the directory you specified. The world save is buried a good bit. I downloaded my game to `~/steam/steamapps/common/palworld`. This puts my world save under `~/steam/steamapps/common/palworld/Pal/Saved/SavedGames/0`. The specific folder that contains the world save is different for everyone, it will be the only directory within the `.../0` directory with a random string of letters and numbers for the name. It won't exist just yet as we have not started up the server at all yet.

All the configuration for the server is done within the `/Saved/Config/LinuxServer/PalWorldSettings.ini` file. There is another `.ini` file in the root directory of your game files with the same name. Do not make changes to that file, any changes you make to that file will not be saved. I would recommend that you set both a `ServerPassword` and `AdminPassword`. They can be found toward the end of the line, along with the `ServerName` and `ServerDescription`.

> All the different configuration options reside on the same __single__ line. For ease of editing through the terminal, if you are using `nano`, use the "_Esc + $_" shortcut to enable soft line wrapping. Meaning press "_Esc_" and let go, hold "_Shift_" and press "_4_".
{: .prompt-info }

To run the server ensure the `PalServer.sh` file in the root directory your game files is executable with `chmod 744 PalServer.sh`. If you run the game server now from your terminal session, you will not be able to interact with that system in any way from that same window and the window will need to remain open. We can solve those issues using __systemd__.

## systemd

Setting up systemd is relatively straight forward as we have already been given the start up script for the game. You will need sudo privileges to create the __systemd__ service. Create an empty file within the `/etc/systemd/system` directory, name it what you like so long as it ends is `.service`. Open it with your preferred text editor, for example `sudo nano /etc/systemd/system/palworld.service`. Then copy the below content to the file:

```bash
[Unit]
Description=PalWorld service
Wants=network.target
After=syslog.target network-online.target

[Service]
Type=simple
Restart=on-failure
RestartSec=10
User=USER
WorkingDirectory=/home/USER/steam/steamapps/common/palworld/
ExecStart=/bin/sh /home/USER/steam/steamapps/common/palworld/PalServer.sh

[Install]
WantedBy=multi-user.target
```
{: .file="/etc/systemd/system/palworld.service}

If you followed along and used the same file paths I did you only need to change every instance of "_USER_" with the name of the user account you want to run the server. Otherwise change the file paths accordingly. Once the file is sorted and contains the correct paths allow systemd to manage it with `sudo systemctl enable palworld.service` and start it with, `sudo systemctl start palworld`. When you need to update the game files stop the server first with `sudo systemctl stop palworld`, move to the directory containing the script from earlier. Run it with `./update-palworld`, upon completion restart the server with `sudo systemctl start palworld`.

## Final Touches

You are just about ready to connect and play with your friends! The last few things you will need to do to connect are:

* Open port `8211` on your servers firewall (if the firewall is enabled)
* Port forward the same port through your gateway/router to your server
* Give the public Ip assigned to your machine (if hosted in the cloud) or the public Ip of your home networks gateway (if hosted locally) to players
* Enter that Ip address into the correct field on Palworld and you are all set!

## References

[steamcmd](https://developer.valvesoftware.com/wiki/SteamCMD)

[Palworld Dedicated Server](https://docs.palworldgame.com/getting-started/deploy-dedicated-server/)

[systemd](https://systemd.io/)
