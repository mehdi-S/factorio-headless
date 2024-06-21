
# factorio-headless

A small, straightforward containerized Factorio server that supports both interactive and non-interactive server commands. Ideal for local and hosted servers, it is simple, low-configuration, and highly customizable.

Source image [Docker Hub](https://hub.docker.com/repository/docker/mehdis/factorio-headless/general)
### ❗IMPORTANT❗ this is a fork from [6davids image](https://hub.docker.com/r/6davids/docker-factorio)

> This fork is designed to address errors found in the main project:
>  1. The URL being used doesn't download the latest Factorio server build; instead, it oddly downloads version 1.1.57
>  2. The README doesn't mention that you need to prepend `./saves/` to your save name in the last-game text file.
>  3. It doesn't explain how to run the container through Docker Compose if you use more advanced tools like
> [Portainer](https://docs.portainer.io/start/install-ce/server/docker/linux)

# Start the server
### · Run With Custom Saved Games :

By default the container creates a new game and then uses it from then on. If you'd like to use your own games, create two **binds** when running the container:

-   `/app/factorio/saves/`
-   `/app/factorio/last-game.txt`

Drop your save files in the  `saves/`  directory, and then enter the name of the save file you'd to use into  `last-game.txt`.

⚠️ it must be **prefixed** by `./saves/` ⚠️

##### Example:
```
cp my-cool-saved-game.zip ~/factorio-stuffs/saved-games/ 
echo "./saves/my-cool-saved-game.zip" > ~/factorio-stuffs/last-game.txt 
```
##### Running:
Start the container detached, with ports mounted, and the two binds mentioned above
**via docker-compose if you're cool :** 
```
services:
  factorio-headless:
    image: mehdis/factorio-headless:1.1
    container_name: factorio-headless
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Etc/UTC
    volumes:
      - type: bind
        source: /docker/factorio-headless/saves
        target: /app/saves
      - type: bind
        source: /docker/factorio-headless/last-game.txt
        target: /app/last-game.txt
    ports:
      - 34197:34197/udp
    restart: unless-stopped
```
**Or via docker run ...** 😒
```
docker run --name factorio-headless -it \
-d \
-p 34197:34197/udp \
--mount type=bind,source=~/factorio-stuffs/saved-games,target=/app/factorio/saves \
--mount type=bind,source=~/factorio-stuffs/last-game.txt,target=/app/factorio/last-game.txt \
mehdis/factorio-headless
```

### · Custom Settings

Settings can be managed the same way that saved games are. Mount files in the default locations to populate the server configs.

# Commands

### · Server Commands

##### Interactive:

*Hop into the container*
```
docker exec -it factorio-headless bash
```
*Start Interactive Script*
```
./interact.sh
```
```
 250.157 Info ServerMultiplayerManager.cpp:943: updateTick(15117) received stateChanged peerID(1) oldState(ConnectedDownloadingMap) newState(ConnectedLoadingMap)
 250.341 Info ServerMultiplayerManager.cpp:943: updateTick(15127) received stateChanged peerID(1) oldState(ConnectedLoadingMap) newState(TryingToCatchUp)
 250.342 Info ServerMultiplayerManager.cpp:943: updateTick(15127) received stateChanged peerID(1) oldState(TryingToCatchUp) newState(WaitingForCommandToStartSendingTickClosures)
 250.355 Info GameActionHandler.cpp:4996: UpdateTick (15127) processed PlayerJoinGame peerID(1) playerIndex(0) mode(connect) 
 250.407 Info ServerMultiplayerManager.cpp:943: updateTick(15131) received stateChanged peerID(1) oldState(WaitingForCommandToStartSendingTickClosures) newState(InGame)
2024-04-09 06:08:43 [JOIN] bardella joined the game
sup
2024-04-09 06:08:51 [CHAT] <server>: sup
2024-04-09 06:09:46 [CHAT] bardella: lol
/kick bardella reason
```

##### Non-Interactive (for scripts or one-offs):
*Hop into the container*
```
docker exec -it factorio-headless bash
```
*Send commands to the server*
```
root@factorio-headless:/app ./run-command.sh "sup"
2024-04-09 17:03:11 [CHAT] <server>: sup
root@factorio-headless:/app ./run-command.sh "/kick bardella reason"
2022-04-09 17:03:24 [KICK] bardella was kicked by <server>. Reason: reason.
1352.490 Info ServerMultiplayerManager.cpp:1061: Disconnect notification for peer (1)
```

##### Or, if you don't need to see the command output:
*Hop into the container*
```
docker exec -it factorio-headless bash
```
*Send commands to the server*
```
root@factorio-headless:/app echo "sup" > ./factorio-server-fifo
root@factorio-headless:/app echo "/kick bardella reason" > ./factorio-server-fifo
```

### · Starting, Stopping, Restarting the Server

The image comes with a few scripts to help managing server execution.

*Hop into the container*
```
docker exec -it factorio-headless bash
```
*Stop the factorio server*
```
root@factorio-headless:/app ./stop-server.sh 
Stopping Factorio Server...
Done!
```
*Start server again after stopping it.*
```
root@factorio-headless:/app ./start-server.sh 
Starting Factorio Server...
Done!
```
*Cycle the server.  Useful after config changes.*
```
root@factorio-headless:/app ./restart-server.sh 
Stopping Factorio Server...
Done!
Starting Factorio Server...
Done!
```

# Testing

### · Building the image:
```
docker build -t factorio-headless .
```

### · Test Running:

> prefer docker-compose, those docker run command are just for testing purposes
##### On x86 machines:

```
docker run --name factorio-headless -itp 34197:34197/udp mehdis/factorio-headless
```
*ctrl-c kill the server*
```
docker run --name factorio-headless -itdp 34197:34197/udp mehdis/factorio-headless
```
*runs detached*

##### On m1 Macs:
```
docker run --name factorio-headless -itp 34197:34197/udp --platform linux/amd64 mehdis/factorio-headless
```
*ctrl-c kill the server*
```
docker run --name factorio-headless -itdp 34197:34197/udp --platform linux/amd64 mehdis/factorio-headless
```
*runs detached*

# Credits
### Thank you once again, *6davids*, for providing these useful yet incomplete resources, which have now been improved.
