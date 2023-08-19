# Install multiple Octoprint instances on Raspberry Pi4 (docker)
*August 2023*

## Install Raspbery Pi OS
1. Install image on SD Card
1. Download and install [Raspberry Pi Imager](https://www.raspberrypi.com/software/)
1. As OS select "RaspberyPI OS (other)" -> "RaspberyOS Lite (32-bit)"
1. Press "Advance options" button and
1. Enable SSH (password)
1. Configure wireless LAN
1. Press Save
1. Press Write
1. Install SD card into RaspberryPI
1. Login to SSH using credentials

## Install Docker
### Set up the repository
```bash
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/raspbian/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
echo \
"deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/raspbian \
"$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

### Install Docker
```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
### Verify Docker
```bash
sudo docker run hello-world
```

## Set aliases for printers
1. Physically connect first printer over USB
1. List connected USB devices
```bash
lsusb
```
You will see one printer path something like 'Bus 001 Device 003: ID **2c99**:**0002** Prusa Original Prusa i3 MK3'
1. Remember ID value, for instance "**2c99**:**0002**" (they will be needed below)
1. Create new UDEV rule
```bash
sudo nano /etc/udev/rules.d/a-usb.rules
```
1. Insert such text, but replace values with yours
```bash
SUBSYSTEM=="tty", ATTRS{idVendor}=="2c99", ATTRS{idProduct}=="0002", SYMLINK+="printer_prusa_1"
                                    ^^^^                      ^^^^                    ^^^^^^^^
```
1. Save file (CTRL+O ENTER)
1. Disconnect first printer
1. Repeat for each printer and make sure each SYMLINK is unique (printer_prusa_1, printer_prusa_2)
1. Reboot UDEV
```bash
sudo udevadm trigger
```
1. Verify connected aliases
```bash
ls /dev/printer_
```

## Run Docker container
1. Install docker-compose.yml
```bash
mkdir octoprint
cd octoprint
nano docker-compose.yml
```
1. Paste [`docker-compose.yml`](docker-compose.yml) content
1. Uncomment services for other printers (if needed)
1. Save file (CTRL+O ENTER)
```bash
sudo docker compose up -d
```

## Verify
11. Open `http://<raspberry_ip>:8001` or 8002, 8003, 8004
12. Configure Octoprint for MK3S https://help.prusa3d.com/article/octoprint-configuration-and-install_2182
