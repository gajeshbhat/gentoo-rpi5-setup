# The Custom MOTD & Performance Dashboard
You can setup a custom login greeting with useful information about the system status using `/etc/profile.d/mymotd.sh`. I have used `fastfetch` and my custom script `pi-status` to print this information

## Setup
Update your Package Tree and perform a deep world update (to run more efficient updates specific to your h/w)

```
# Sync the package database
sudo emaint sync -r gentoo

# Perform the deep world update
# Since you have 4 Cortex-A76 cores, use -j5
sudo emerge --ask --verbose --update --deep --newuse @world
```
Since neofetch was discontinued and removed from the Gentoo tree in late 2024, we will use Fastfetch, its high-performance C-based successor.

```
# Install fastfetch
sudo emerge --ask app-misc/fastfetch
```
## Create the Telemetry Script
```
# Open an empty file in nano
sudo nano /usr/local/bin/pi-status
```
Paste this code:

```
#!/bin/bash
BLUE='\033[1;34m'
NC='\033[0m'

fastfetch
echo -e "${BLUE}--- Pi 5 Performance ---${NC}"
# Pulls the 6-second boot data from systemd
BOOT=$(systemd-analyze | awk '/Startup finished/ {print $4 " (kernel) + " $7 " (userspace) = " $10}')
TEMP=$(echo "scale=1; $(cat /sys/class/thermal/thermal_zone0/temp)/1000" | bc)

echo -e "🚀 Boot: $BOOT"
echo -e "🌡️  Temp: ${TEMP}°C"
echo -e "${BLUE}------------------------${NC}"
```
Enable the Login Trigger:

```
sudo chmod +x /usr/local/bin/pi-status
sudo ln -s /usr/local/bin/pi-status /etc/profile.d/mymotd.sh
```

Now you can logout and login and you will be able to see a login greeting with system information. You can also invoke it using the command line using the command `pi-status`
