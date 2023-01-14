Orange Pi 5, Ubuntu Server using Docker for services.

### Services
- homeassistant
- zigbee2MQTT (ZigBee handler/server)
- mosquitto (MQTT broker/server, needed by Zigbee2MQTT)
- scrypted (for unsupported HomeKit Secure Video cameras)
- scrypted-watchtower ( scrypted watcher / updater )


### Initial setup

## Fix APT from ‘default’ mirrors
```
echo "deb http://ports.ubuntu.com/ubuntu-ports jammy main restricted universe multiverse
deb http://ports.ubuntu.com/ubuntu-ports jammy-updates main restricted universe multiverse
deb http://ports.ubuntu.com/ubuntu-ports jammy-backports main restricted universe multiverse
deb http://ports.ubuntu.com/ubuntu-ports jammy-security main restricted universe multiverse
deb http://archive.canonical.com/ubuntu jammy partner" | sudo tee /etc/apt/sources.list > /dev/null
```

## Add new keyring and repository for docker
```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

## Update & Upgrade system via APT
```
sudo apt update
sudo apt full-upgrade
```

## Setup timezone
```
sudo tzselect
echo $TZ | sudo tee /etc/timezone > /dev/null

```

## Go through the builtin OrangePi config utility, set hostname, keyb. etc.
```
sudo orangepi-config
```

## Disable Auto-Login of orangepi user
```
sudo rm /usr/share/systemd/getty@.service.d/override.conf
sudo rm /usr/lib/systemd/system/serial-getty@.service.d/override.conf
sudo systemctl daemon-reload
```

## Add a new user & append to groups
```
export NEWUSER=MyUserName
sudo adduser $NEWUSER
sudo usermod -a -G tty,disk,dialout,sudo,audio,video,plugdev,games,users,systemd-journal,input,netdev,docker $NEWUSER
```

## Add newuser to sudoers to req. no password (optional)
```
echo "$(NEWUSER) ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/90-Users > /dev/null
```

## Apt install sanity check
```
apt install wiringpi ca-certificates curl gnupg lsb-release docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

## Enable docker on boot
```
sudo systemctl enable docker
```

# Reboot and login as new user, and let's continue.
```
sudo reboot
```

## Clean up in /root & Delete orangepi default user
```
sudo -i
cd /root
deluser orangepi
rm -fr /home/orangepi
rm -fr .oh-my-zsh .zshrc .local .desktop_autologin .config .lesshst
```

# Docker Setup

## Creating a directory (/opt) to keep all docker config and container settings
```
sudo mkdir -p /opt/
cd /opt
```

## Initial files, we'll be using Portainer.io for Docker WebUI
```
cat <<EOT >> .env
PUID=1001
PGID=1001
TZ=$(cat /etc/timezone)
EOT

cat <<EOT >> docker-compose.yml
version: '3'
services:
  portainer:
    image: portainer/portainer-ce:latest
    container_name: portainer
    environment:
      - TZ=${TZ}
    volumes:
      - /opt/portainer:/data
      - /var/run/docker.sock:/var/run/docker.sock
    network_mode: host
    restart: unless-stopped"
EOT

docker compose up -d
```

## Now got to your browser and goto https://hostname:9443 and setup Portainer

# Portainer

## After setup goto 'stacks' and select '+ add stack'

Remember to make 3 variables in every stack, by pressing '+ add an enviorment variable'
name: PUID, data: 1001
name: PGID, data: 1001
name: TZ, data: (your timezone as above)

## Add data in 'smarthome.yaml' to the webeditor

## Optional Extra Stack: 'extra.yaml'
- homebridge
- nodered

## Optional Media Stack: 'homemedia.yaml'
- Transmission
- Plex Server
