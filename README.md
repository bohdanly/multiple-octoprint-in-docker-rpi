# Install multiple Octoprint instances on Raspberry Pi4 (docker)
*August 2023*

## Install Raspbery Pi OS
- Install image on SD Card
- Download and install [Raspberry Pi Imager](https://www.raspberrypi.com/software/)
- As OS select "RaspberyPI OS (other)" -> "RaspberyOS Lite (32-bit)"
- Press "Advance options" button and
- Enable SSH (password)
- Configure wireless LAN
- Press Save
- Press Write
- Install SD card into RaspberryPI
- Login to SSH using credentials

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
- Physically connect first printer over USB
- List connected USB devices
```bash
lsusb
```
You will see one printer path something like 'Bus 001 Device 003: ID **2c99**:**0002** Prusa Original Prusa i3 MK3'
- Remember ID value, for instance "**2c99**:**0002**" (they will be needed below)
- Create new UDEV rule
```bash
sudo nano /etc/udev/rules.d/a-usb.rules
```
- Insert such text, but replace values with yours
```bash
SUBSYSTEM=="tty", ATTRS{idVendor}=="2c99", ATTRS{idProduct}=="0002", SYMLINK+="printer_prusa_1"
                                    ^^^^                      ^^^^                    ^^^^^^^^
```
- Save file (CTRL+O ENTER)
- Disconnect first printer
- Repeat for each printer and make sure each SYMLINK is unique (printer_prusa_1, printer_prusa_2)
- Reboot UDEV
```bash
sudo udevadm trigger
```
- Verify connected aliases
```bash
ls /dev/printer_
```

## Run Docker container
- Install docker-compose.yml
```bash
mkdir octoprint
cd octoprint
nano docker-compose.yml
```
- Paste [`docker-compose.yml`](docker-compose.yml) content
- Uncomment services for other printers (if needed)
- Save file (CTRL+O ENTER)
```bash
sudo docker compose up -d
```

## Verify
- Open `http://<raspberry_ip>:8001` or 8002, 8003, 8004
- Configure Octoprint for MK3S https://help.prusa3d.com/article/octoprint-configuration-and-install_2182
