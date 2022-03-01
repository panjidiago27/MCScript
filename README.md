# MCScript
Minecraft Script

###### as root
apt-get update -y && apt-get install -y make curl gcc git build-essential openjdk-8-jre-headless
useradd -r -m -U -d /opt/minecraft -s /bin/bash minecraft
ufw allow 25565
ufw allow 22
ufw enable

###### as minecraft user
mkdir -p ~/{backups,tools,server}
cd ~/tools && git clone https://github.com/Tiiffi/mcrcon.git
cd ~/tools/mcrcon
gcc -std=gnu99 -Wall -Wextra -Wpedantic -Os -s -fstack-protector-strong -o mcrcon mcrcon.c
./mcrcon -h
cd server
wget https://launcher.mojang.com/v1/objects/c8f83c5655308435b3dcf03c06d9fe8740a77469/server.jar
java -Xmx3072M -Xms2048M -jar server.jar nogui

# eula.txt
#By changing the setting below to TRUE you are indicating your agreement to our EULA (https://account.mojang.com/documents/minecraft_eula).
#Sat Feb 08 00:08:45 CET 2020
eula=true

# server.properties
#Minecraft server properties
#Wed Feb 12 17:39:54 CET 2020
spawn-protection=16
max-tick-time=60000
query.port=25565
generator-settings=
force-gamemode=false
allow-nether=true
enforce-whitelist=false
gamemode=creative
broadcast-console-to-ops=true
enable-query=false
player-idle-timeout=0
difficulty=peaceful
spawn-monsters=true
broadcast-rcon-to-ops=true
op-permission-level=4
pvp=true
snooper-enabled=true
level-type=default
hardcore=false
enable-command-block=true
max-players=5
network-compression-threshold=256
resource-pack-sha1=
max-world-size=29999984
function-permission-level=2
rcon.port=25575
server-port=25565
server-ip=
spawn-npcs=true
allow-flight=false
level-name=Mondo_di_Luca
view-distance=10
resource-pack=
spawn-animals=true
white-list=false
rcon.password=FUNKYPASSWORD
generate-structures=true
max-build-height=256
online-mode=true
level-seed=jfewkjwejwef
use-native-transport=true
prevent-proxy-connections=false
enable-rcon=true
motd=Minecraft Server di Luca ed i suoi amici

# /opt/minecraft/tools/backup.sh
#!/bin/bash
function rcon {
          /opt/minecraft/tools/mcrcon/mcrcon -H 127.0.0.1 -P 25575 -p FUNKYPASSWORD "$1"
  }
rcon "save-off"
rcon "save-all"
tar -cvpzf /opt/minecraft/backups/server-$(date +%F_%R).tar.gz /opt/minecraft/server
rcon "save-on"
find /opt/minecraft/backups/ -type f -mtime +7 -name '*.gz' -delete

###### as minecraft user
chmod +x /opt/minecraft/tools/backup.sh

# crontab
00 01 * * * /opt/minecraft/tools/backup.sh

###### as root

# /etc/systemd/system/minecraft.service
[Unit]
Description=Minecraft Server
After=network.target

[Service]
User=minecraft
Nice=1
KillMode=none
SuccessExitStatus=0 1
ProtectHome=true
ProtectSystem=full
PrivateDevices=true
NoNewPrivileges=true
WorkingDirectory=/opt/minecraft/server
ExecStart=/usr/bin/java -Xmx3072M -Xms2048M -jar server.jar nogui
ExecStop=/opt/minecraft/tools/mcrcon/mcrcon -H 127.0.0.1 -P 25575 -p FUNKYPASSWORD stop

[Install]
WantedBy=multi-user.target

# as root
systemctl daemon-reload
systemctl enable minecraft
