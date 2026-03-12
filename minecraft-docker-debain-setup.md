```bash
sudo apt update
```
```bash
sudo apt upgrade
```
```bash
sudo apt install openssh-server
```
```bash
sudo systemctl enable ssh
```
- - -
```md
ssh user@ip
```
- - -
```bash
sudo nano setup.sh
```
```bash
#!/usr/bin/env bash

SERVER_DIR="/mc"
SERVER_NAME="Minecraft Server"
MOTD="Welcome!"
MC_VERSION="latest"
MAX_MEMORY="6G"

ENABLE_SPARK=true
ENABLE_CHUNKY=true
ENABLE_FAWE=true
ENABLE_HORIZONS=true
ENABLE_BLUEMAP=false
ENABLE_PLAN=false

OPT_EIGENCRAFT_REDSTONE=true
OPT_OPTIMIZE_EXPLOSIONS=true
OPT_NETWORK=true
OPT_CHUNK_LOADING=true
OPT_ANTIXRAY_ENGINE1=true

apt update -y
apt install -y ca-certificates curl gnupg lsb-release wget unzip

install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg
chmod a+r /etc/apt/keyrings/docker.gpg

echo \
"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" \
| tee /etc/apt/sources.list.d/docker.list > /dev/null

apt update -y
apt install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin

mkdir -p "$SERVER_DIR"

cat > "$SERVER_DIR/docker-compose.yml" <<EOF
version: "3.9"
services:
  mc:
    image: itzg/minecraft-server:latest
    container_name: paper
    ports:
      - "25565:25565"
    environment:
      EULA: "TRUE"
      TYPE: PAPER
      VERSION: "$MC_VERSION"
      MEMORY: "$MAX_MEMORY"
      TZ: "Europe/Berlin"
      MOTD: "$MOTD"
      SERVER_NAME: "$SERVER_NAME"
    volumes:
      - $SERVER_DIR/data:/data
    restart: unless-stopped
EOF

docker compose -f "$SERVER_DIR/docker-compose.yml" up -d
sleep 10

PLUGINS_DIR="$SERVER_DIR/data/plugins"
mkdir -p "$PLUGINS_DIR"
cd "$PLUGINS_DIR"

if [ "$ENABLE_SPARK" = true ]; then
wget -q https://github.com/lucko/spark/releases/latest/download/spark-paper.jar -O Spark.jar
fi

if [ "$ENABLE_CHUNKY" = true ]; then
wget -q https://github.com/pop4959/Chunky/releases/latest/download/Chunky.jar -O Chunky.jar
fi

if [ "$ENABLE_FAWE" = true ]; then
wget -q https://ci.athion.net/job/FastAsyncWorldEdit/lastSuccessfulBuild/artifact/build/libs/FastAsyncWorldEdit-Paper-1.21.jar -O FAWE.jar
fi

if [ "$ENABLE_HORIZONS" = true ]; then
wget -q https://download.luminolmc.com/horizons.jar -O Horizons.jar
fi

if [ "$ENABLE_BLUEMAP" = true ]; then
mkdir -p BlueMap
wget -q https://github.com/BlueMap-Minecraft/BlueMap/releases/latest/download/BlueMap-3.15.jar -O BlueMap.jar
fi

if [ "$ENABLE_PLAN" = true ]; then
wget -q https://github.com/plan-player-analytics/Plan/releases/latest/download/Plan.jar -O Plan.jar
fi

docker restart paper
sleep 5

GLOBAL_CFG="$SERVER_DIR/data/paper-global.yml"
SPIGOT_CFG="$SERVER_DIR/data/spigot.yml"

if [ -f "$GLOBAL_CFG" ]; then

[ "$OPT_EIGENCRAFT_REDSTONE" = true ] && \
sed -i 's/use-faster-eigencraft-redstone: false/use-faster-eigencraft-redstone: true/' "$GLOBAL_CFG"

[ "$OPT_OPTIMIZE_EXPLOSIONS" = true ] && \
sed -i 's/optimize-explosions: false/optimize-explosions: true/' "$GLOBAL_CFG"

[ "$OPT_ANTIXRAY_ENGINE1" = true ] && \
sed -i 's/enabled: false/enabled: true/' "$GLOBAL_CFG" && \
sed -i 's/engine-mode: .*/engine-mode: 1/' "$GLOBAL_CFG"
fi

if [ -f "$SPIGOT_CFG" ]; then
if [ "$OPT_NETWORK" = true ]; then
sed -i 's/bungee-online-mode: true/bungee-online-mode: true/' "$SPIGOT_CFG"
fi
fi

echo "SETUP COMPLETE"
```
```bash
sudo chmod +x setup.sh
```
```bash
sudo ./setup.sh
```
