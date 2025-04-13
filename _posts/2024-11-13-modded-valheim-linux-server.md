---
layout: post
title: Modded Valheim Linux Server
date: 2024-11-13 20:41:00 +-0000
image: /assets/img/preview/valheim-preview.png
description: Short tutorial to get a Valheim server up and running on Linux.
categories: [Guide,Reminders]
tags: [systemd,bash,games]
---

## Services and Required Knowledge

[__steamcmd__](https://developer.valvesoftware.com/wiki/SteamCMD) is a command line tool required to manage any and all steam games you wish to host locally on a Linux based server, including Valheim. Ensure it is installed before proceeding. If you do not see __steamcmd__ listed in the output of the first command use the second to install it.

> Examples are assuming you have a Debian based Linux distribution using the `apt` package manager. Please use your provided distribution's package manager and its syntax.
{: .prompt-info }

```bash
sudo apt list --installed | grep steamcmd

sudo apt install steamcmd -y
```

Each steam game is identified on steam with a unique identifier called an "_App ID_," or colloquially a, "_Steam ID_." You will need to find the proper ID and pass that into __steamcmd__ when requesting to download or update the game. Be careful, as you will want to ensure you are using the Steam ID for Valheim's dedicated server version of the game, not the standard game installed on your gaming PC. You can find any Steam ID you need simply from searching google, or you can use the [steamdb](https://steamdb.info/) website. Valheim Server's Steam ID is: `896660`.

Once you have __steamcmd__ installed and have the necessary Steam ID you can run the command below to download the game files:

> For ease of use later with systemd setup use a path with __NO__ spaces or special characters.
{: .prompt-warning }

```bash
steamcmd +login anonymous +force_install_dir /PATH/YOU/WANT/GAME/FILES +app_update 896660 validate +exit
```

Once the game files have been downloaded save this command for later within a script. It can be run at a later time to update the game whenever the Valheim team releases a new version. I saved mine under `~/bin/update-valheim`. Mark your file as executable with `chmod` and you can stop there if you like. If you leave out the file extension, as I have done, you can add the script to your path and run the script as if it were a built-in command in your shell. Add this to the bottom of your `.bashrc` file:

```bash
# adding custom scripts within the ~/bin directory to function similar to built-in commands
export PATH=$PATH:~/bin
```
{: file="~/.bashrc" }

If followed exactly you can type `update-valheim` into your terminal from anywhere and the __steamcmd__ command will run to update the game.

## Important Files and File Structure

All of your important game files will be stored under the path you specify within the initial command for the download. e.g. Mine are stored under `~/steam/steamapps/common/valheim`, this holds all important binaries and other assets required to run the game. The save files for your world, along with the list of in game admins and banned players are found in: `~/.config/unity3d/IronGate/Valheim`.

The file used to start the server comes along with the download of the game, `start_server.sh`. The file itself has instructions written inside in the form of comments. Open up the file in your preferred text editing software and follow the instructions. It is highly recommended that you save a copy of the file locally, or in a separate folder on the server, incase Steam decides it wants to overwrite the file. I've never ran into that issue but it is always better to be safe than sorry. 

Should you want to install mods to improve upon and change the vanilla Valheim experience in anyway; that will require the installation of __BepInEx__ on top of your Valheim install. You can download the files from either the __BepInEx__ [website](https://docs.bepinex.dev/index.html), or from the [GtiHub](https://github.com/BepInEx/BepInEx/releases) page for the project. I have found the GitHub page more straight forward to follow. Simply download the `.zip` file designated for your operating system and subsequent architecture. After downloading, unzip the file and copy the contents to the root of your game installation. Following my own installation that is `~/steam/steamapps/common/valheim`. I do not remember if BepInEx comes with a pre-configured start up script. In the case it does not I have my script below as well.

```bash
#!/usr/bin/env bash
# BepInEx-specific settings
# NOTE: Do not edit unless you know what you are doing!
####
export DOORSTOP_ENABLE=TRUE
export DOORSTOP_INVOKE_DLL_PATH=./BepInEx/core/BepInEx.Preloader.dll
export DOORSTOP_CORLIB_OVERRIDE_PATH=./unstripped_corlib

export LD_LIBRARY_PATH="./doorstop_libs:$LD_LIBRARY_PATH"
export LD_PRELOAD="libdoorstop_x64.so:$LD_PRELOAD"
####


export LD_LIBRARY_PATH="./linux64:$LD_LIBRARY_PATH"
export SteamAppId=892970

echo "Starting server PRESS CTRL-C to exit"

# Tip: Make a local copy of this script to avoid it being overwritten by steam.
# NOTE: Minimum password length is 5 characters & Password cant be in the server name.
# NOTE: You need to make sure the ports 2456-2458 are being forwarded to your server through your local router/firewall.
exec ./valheim_server.x86_64 -name "NAME OF THE SERVER DISPLAYED IN GAME" -port 2456 -world "NAME FOR WORLD SAVE FILE" -password "PASSWORD USED IN GAME"
```
{: file="~/steam/steamapps/common/valheim/start_server_bepinex.sh" }

> At the time of writing the steps for installation when downloading BepInEx from their website seem to differ slightly. I have never followed this route. If you wish too please follow the documentation listed there.
{: .prompt-info }

To add mods to your server the configuration files can be found under `~/steam/steamapps/common/valheim/BepInEx`. In here you will find a `plugins` folder for storing specific binaries and additional files required to get mods up and running. Each individual mod will need its own folder to hold those files required for the mod to function. Using my installation as an example the path will look something like this: `~/steam/steamapps/common/valheim/BepInEx/plugins/FOLDER-FOR-MOD/MOD-FILE(s)`. Typically every mod is packaged as a `.zip` file and taking the output folder from unzipping it and adding it to the `plugins` folder will suffice. 

## systemd

Running the server through __systemd__ will allow you to easily keep the game up and running 24/7. Starting the server straight from your command prompt introduces some issues. Namely, you are stuck leaving your terminal session open, and will not be able to interact with your server in any other capacity, assuming your game server is not running on a Linux desktop. You will need to create a __systemd__ unit file and drop it into the `/etc/systemd/system` directory. Below is what my file looks like.

```
[Unit]
Description=Valheim Modded service
Wants=network.target
After=syslog.target network-online.target

[Service]
Type=simple
Restart=on-failure
RestartSec=10
User=USER
WorkingDirectory=/home/USER/steam/steamapps/common/valheim/
ExecStart=/bin/sh /home/USER/steam/steamapps/common/valheim/start_server_bepinex.sh

[Install]
WantedBy=multi-user.target
```
{: file="/etc/systemd/system/valheim_modded.service"}

If you have followed my installation exactly make sure to replace all instances of "_USER_" with the name of your user on the Linux server. Other wise be sure to to change the "_WorkingDirectory_" with the path to your game installation, and the "_ExecStart_" path to the path of your start up script. Afterwards you can run `sudo systemctl enable valheim_modded`. Use `sudo systemctl start valheim_modded` to start up the server for the first time. After the first start you won't ever have to do that again unless you stop the server manually with `sudo systemctl stop valheim_modded`. To check if the service is running properly use `sudo systemctl status valheim_modded`. If for what ever reason your server fails to start, you can use `journalctl` to see the logs within the valheim service. i.e. `sudo journalctl -u valheim_modded -n 20` will show you the last 20 lines worth of the log. 

## Final Touches

You are just about ready to connect and play with your friends! The last few things you will need to do to connect are:

* Open ports `2456-2458` on your servers firewall (if the firewall is enabled)
* Port forward those same ports through your gateway/router to your server
* Give the public Ip assigned to your machine (if hosted in the cloud) or the public Ip of your home networks gateway (if hosted locally) to players
* Enter that Ip address into the correct field on Valheim and you are all set! 

### Updating the Game Server

When it comes time to update there is not a whole lot that needs to be done.

* First stop the server with `sudo systemctl stop valheim_modded`
* Next replace any mods that require them in your `~/steam/steamapps/common/valheim/BepInEx/plugins` folder
* Then run `update-valheim`
* Lastly, restart the server with `sudo systemctl start valheim_modded`

## References

[Harre.dev Valheim Server on Linux](https://harre.dev/blog/valheim-linux-server/?utm_source=harrewijnen-net)

[BepInEx](https://docs.bepinex.dev/index.html)

[steamcmd](https://developer.valvesoftware.com/wiki/SteamCMD)

[systemd](https://systemd.io/)