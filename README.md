# MCScript
Minecraft Script

###### as root
apt-get update -y && apt-get install -y make curl gcc git build-essential openjdk-8-jre-headless <br>
useradd -r -m -U -d /opt/minecraft -s /bin/bash minecraft <br>
ufw allow 25565 <br>
ufw allow 22 <br>
ufw enable <br>

###### as minecraft user
mkdir -p ~/{backups,tools,server} <br>
cd ~/tools && git clone https://github.com/Tiiffi/mcrcon.git <br>
cd ~/tools/mcrcon <br>
gcc -std=gnu99 -Wall -Wextra -Wpedantic -Os -s -fstack-protector-strong -o mcrcon mcrcon.c <br>
./mcrcon -h <br>
cd server <br>
wget https://launcher.mojang.com/v1/objects/c8f83c5655308435b3dcf03c06d9fe8740a77469/server.jar <br>
java -Xmx3072M -Xms2048M -jar server.jar nogui <br>

# eula.txt
#By changing the setting below to TRUE you are indicating your agreement to our EULA (https://account.mojang.com/documents/minecraft_eula). <br>
#Sat Feb 08 00:08:45 CET 2020 <br>
eula=true <br>

# server.properties
#Minecraft server properties <br>
#Wed Feb 12 17:39:54 CET 2020 <br>
spawn-protection=16 <br>
max-tick-time=60000 <br>
query.port=25565 <br>
generator-settings= <br>
force-gamemode=false <br>
allow-nether=true <br>
enforce-whitelist=false <br>
gamemode=creative <br>
broadcast-console-to-ops=true <br>
enable-query=false <br>
player-idle-timeout=0 <br>
difficulty=peaceful <br>
spawn-monsters=true <br>
broadcast-rcon-to-ops=true <br>
op-permission-level=4 <br>
pvp=true <br>
snooper-enabled=true <br>
level-type=default <br>
hardcore=false <br>
enable-command-block=true <br>
max-players=5 <br>
network-compression-threshold=256 <br>
resource-pack-sha1= <br>
max-world-size=29999984 <br>
function-permission-level=2 <br>
rcon.port=25575 <br>
server-port=25565 <br>
server-ip= <br>
spawn-npcs=true <br>
allow-flight=false <br>
level-name=Mondo_di_Luca <br>
view-distance=10 <br>
resource-pack= <br>
spawn-animals=true <br>
white-list=false <br>
rcon.password=FUNKYPASSWORD <br>
generate-structures=true <br>
max-build-height=256 <br>
online-mode=true <br>
level-seed=jfewkjwejwef <br>
use-native-transport=true <br>
prevent-proxy-connections=false <br>
enable-rcon=true <br>
motd=Minecraft Server di Luca ed i suoi amici <br>

# /opt/minecraft/tools/backup.sh
#!/bin/bash <br>
function rcon { <br>
          /opt/minecraft/tools/mcrcon/mcrcon -H 127.0.0.1 -P 25575 -p FUNKYPASSWORD "$1" <br>
  } <br>
rcon "save-off" <br>
rcon "save-all" <br>
tar -cvpzf /opt/minecraft/backups/server-$(date +%F_%R).tar.gz /opt/minecraft/server <br>
rcon "save-on" <br>
find /opt/minecraft/backups/ -type f -mtime +7 -name '*.gz' -delete <br>

###### as minecraft user
chmod +x /opt/minecraft/tools/backup.sh <br>

# crontab
00 01 * * * /opt/minecraft/tools/backup.sh <br>

###### as root

# /etc/systemd/system/minecraft.service
[Unit] <br>
Description=Minecraft Server <br>
After=network.target <br>

[Service]
User=minecraft <br>
Nice=1 <br>
KillMode=none <br>
SuccessExitStatus=0 1 <br>
ProtectHome=true <br>
ProtectSystem=full <br>
PrivateDevices=true <br>
NoNewPrivileges=true <br>
WorkingDirectory=/opt/minecraft/server <br>
ExecStart=/usr/bin/java -Xmx3072M -Xms2048M -jar server.jar nogui <br>
ExecStop=/opt/minecraft/tools/mcrcon/mcrcon -H 127.0.0.1 -P 25575 -p FUNKYPASSWORD stop <br>

[Install] <br>
WantedBy=multi-user.target <br>

# as root
systemctl daemon-reload <br>
systemctl enable minecraft <br>
