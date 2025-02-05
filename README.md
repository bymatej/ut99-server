# ut99-server
This is just a fork of another project. See `Frok` for details.

A dockerfile for a fully functional and easy configurable Unreal Tournament 99 server.
This image is based on the original linux server 436 with all four bonus packs and the UTPGPatch451 patch for linux.
It also includes some famous maps and mutators.
All data can be adjusted with a `named volume` (host bind won't work) (see `Usage` for details).

# Usage
Just run the docker image with the following command:
```
docker run --name ut99 -p 5580:5580 -p 7777:7777/udp -p 7778:7778/udp -v ut99-data:/ut-data bymatej/ut99-server:latest
```
This will create and run the container, exposing the web-admin under port 5580 and the game under 7777.
It will also create a docker volume named `ut99-data` which contains all the ini files and non-standard maps and mods.
You can even add your own maps and mods there or edit all the ini files in that volume and restart the server for the new files or configs.
This basically works by having this files in this volume and on start of the server, all files in the volume are symlinked into the corresponding ut-server folder.

## Environment Variables
| Variable | Mandatory | Description |
| -------- | --------- | ----------- |
| UT_SERVERURL | Yes | This is the default uri for the server startup. By default, it looks like this `CTF-Face?game=BotPack.CTFGame?mutator=BotPack.SniperArena,MapVoteLAv2.BDBMapVote,FlagAnnouncementsV2.FlagAnnouncements,ZeroPingPlus103.ZP_SniperArena,KickIdlePlayers2.KickIdlePlayers2` |
| UT_SERVERNAME | No | If this variable is set, it will always override the server name in `UnrealTournament.ini` with this on startup. |
| UT_ADMINNAME | No | If this variable is set, it will always override the admin name in `UnrealTournament.ini` with this on startup. |
| UT_ADMINEMAIL | No | If this variable is set, it will always override the admin email in `UnrealTournament.ini` with this on startup. |
| UT_MOTD1 | No | If this variable is set, it will always override the MOTD1 in `UnrealTournament.ini` with this on startup. |
| UT_DOUPLINK | No | If this variable is set, it will always override the DoUpLink in `UnrealTournament.ini` with this on startup. Default is `true`.|
| UT_ADMINPWD | No | If this variable is set, it will always override the admin password in `UnrealTournament.ini` with this on startup. |
| UT_GAMEPWD | No | If this variable is set, it will always override the game password in `UnrealTournament.ini` with this on startup. |
| UT_WEBADMINUSER | No | If this variable is set, it will always override the web admin username in `UnrealTournament.ini` with this on startup. |
| UT_WEBADMINPWD | No | If this variable is set, it will always override the web admin password in `UnrealTournament.ini` with this on startup. |

## Volumes
As mentioned above, there is one named volume that should point to `/ut-data` in the container.
This folder contains all important folders (`Maps`, `Music`, `Sounds`, `System`, `Textures`) where any non-standard files can be placed and will be loaded on the next restart of the server/container.
Also all ini files can be found there in the `System` folder so they can be adjusted as well.

## How I use it on my server
I want a custom mountpoint so that I can easily navigate to my files and maintain my server. I mount the volume a little bit differently. Also, I specify some of the environment variables right in the run command. This is how it looks like:
- I create a directory for the mount point (for volume):
  ```
  mkdir -p /home/username/games/ut99/public
  ``` 
- I run my docker run command: 
  ```
  docker run -d -e UT_SERVERNAME='My Cool UT server' -e UT_ADMINNAME='myadminusername' -e UT_ADMINEMAIL='games@bymatej.com' -e UT_MOTD1='The best message of the day ever' -e UT_ADMINPWD='MyCoolAdminPass' -e UT_WEBADMINUSER='mywebadminuser' -e UT_WEBADMINPWD='MyCoolWebAdminPass' --name ut99_public -p 5580:5580 -p 7777:7777/udp -p 7778:7778/udp --mount type=volume,source=ut99-data-public,target=/ut-data,volume-driver=local,volume-opt=type=none,volume-opt=o=bind,volume-opt=device=/home/username/games/ut99/public bymatej/ut99-server:latest
  ```


# Included Mods and Mutators

## CustomCrossHairScale
This mod is loaded permanently. It allows to scale the crosshair as it might be too big on some resolutions.
Any player can just go to their console (tab) and execute the following command:
`mutate ch_scale 1`

## FlagAnnouncementsV2
This mod is added as a mutator. So it must be added to the mutators list to work.
When this mod is enabled, there is a voice that tells if and which flag is taken/dropped/returned.
Further configuration can be done in the `System/FlagAnnouncements.ini` file.

## KickIdlePlayers2
This mod is added as a mutator. So it must be added to the mutators list to work.
When this mod is enabled, inactive users will be kicked from the server.
Further configuration can be done in the `System/KickIdlePlayers2.ini` file.

## MapVoteLAv2
This mod is added as a mutator. So it must be added to the mutators list to work.
When this mode is enabled, a map vote / kick screen will come after each map so the users can vote for the next map.
Further configuration can be done in the `System/MapVoteLA.ini` file.

## NoSelfDamagev03
This mod is added as a mutator. So it must be added to the mutators list to work.
When this mod is enabled, self damage (for example from rocket launchers) can be disabled.
Further configuration can be done in the `System/NoSelfDamage.ini` file.

## WhoPushedMe
This mod is added as a mutator. So it must be added to the mutators list to work.
When this mode is enabled, detects if someone is killed because someone else pushed/hit them and they fell to their death and grants the killer the corresponding points (or negative points when a teammate is killed).

## ZeroPingPlus103
This mod is added as a mutator. So it must be added to the mutators list to work.
When this mod is enabled, the clientside calculates if a hit was a hit or not and tells this the server, effectively leading to 0 ping.

# Fork
## Original author
Original author of this project is Roemer.
- His Git Repo: https://github.com/Roemer/ut99-server
- His DockerHub repo: https://hub.docker.com/r/roemer/ut99-server
- His posts on ut99 forums: https://ut99.org/viewtopic.php?t=13731
- Another useful post: https://ut99.org/viewtopic.php?t=167

All credits for creating this project goes to him!

## WTF? -> Why The Fork?

### Reason 1
I wanted to enable this portion of configuration in the prepare script:

    [IpServer.UdpServerUplink]
    DoUpLink=True
    UpdateMinutes=1
    MasterServerPort=27900

  Also, I wanted to make the `DoUpLink` property configurable in case you don't want your server to be visible in the server list in the game.

### Reason 2
I wanted to add some maps, skins and mods more easily. By default, the mount point for the docker volumes is at `/var/lib/docker/volumes/`. I wanted to change it. Turns out it is possible by mounting the volume with some extra attributes. Check out the "How I use it on my server" section.
Basically, instead of `-v ut99-data-public:/ut-data` i use this `-mount type=volume,source=ut99-data-public,target=/ut-data,volume-driver=local,volume-opt=type=none,volume-opt=o=bind,volume-opt=device=/home/username/games/ut99/public`. The only "downside" to this is that I need to create the `/home/username/games/ut99/public` before executing the command. The only "upside" is that now I have the mount point in my home directory.
It is a slight difference, but it makes me happy. :)

Sources: 
- https://stackoverflow.com/questions/39496564/docker-volume-custom-mount-point
- https://docs.docker.com/storage/volumes/#choose-the--v-or---mount-flag
