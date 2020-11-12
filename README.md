# Raspberry Pi Setup Guide

## Setup Pi Without Monitor or Keyboard

1. Download Image
2. Open Disk Utility on Mac OS and click on “Show all Devices”
3. Format SD Card with FAT 32
4. sudo diskutil unmountDisk /dev/rdisk2
5. sudo dd if=2020-08-20-raspios-buster-arm64-lite.img of=/dev/rdisk2 bs=32m
6. touch a file named ssh and copy to boot drive
7. touch a file named wpa_supplicant.conf and add contents below and copy to boot drive 
    ```
   country=US
   ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
   update_config=1
   
   network={
       ssid="NETWORK-NAME"
       psk="NETWORK-PASSWORD"
   }
   ```
8. sudo diskutil eject /dev/rdisk2
9. Login to pi using ssh pi@ipaddress

## Things to do after first boot

### Add new user with root privs
1. Change password on pi account by typing passwd
2. sudo adduser xxx
3. Enter password for new account
4. sudo usermod -a -G adm,dialout,cdrom,sudo,audio,video,plugdev,games,users,input,netdev,gpio,i2c,spi xxxx

### Change hostname
1. sudo vi /etc/hostname
2. sudo vi /etc/hosts

### Remove Pi Account
1. Login as new user
2. sudo pkill -u pi
3. sudo deluser pi

### Update Pi
#### Update System's Package List
1. sudo apt update

#### Upgrade all installed packages to their latest versions
1. sudo apt full-upgrade

### Create ssh keys

## Add External Storage and NFS Mount
### Format USB Drive
1. sudo mkfs.vfat /dev/sda1 -n WUDRIVE
### Mount Drive
2. create folder where mount drive: sudo mkdir /mnt/xxxx
3. sudo mount /dev/sda1 /mnt/xxxx

### Ensure Drive is mounted after reboot
1. Get UUID of the disk partion.

    ```sudo blkid```
2. Edit /etc/fstab with info below. If filesystem is FAT add ,umask=000 to end of nofail. This will allow all users read/write access to every file.

    ```UUID=8AAE-2AA7 /mnt/wudrive vfat defaults,auto,users,rw,nofail,umask=000 0 0```
## Install and Configure Transmission
### Install Transmission
1. sudo apt install -y transmission-daemon
2. sudo service transmission-daemon stop

### Allow Transmission to be managed remotely
1. sudo vi /etc/transmission-daemon/settings.json
    ```
    “rpc-whitelist”: “127.0.0.1,192.168.*.*”,
    “rpc-whitelist-enabled”: true,
    ```
### Setup Transmission User Name and Password
     ```
     “rpc-username”: “xxxxx”,
     “rpc-password”: "xxxxx",
     ```
  
### Change Transmission download-dir
     ```
     "download-dir": "/mnt/xxxx"
     ```
### Disable login/pass for Transmission GUI

### Start Transmission
1. sudo service transmission-daemon start

## Install and Configure PIA

### Install OpenVPN
1. sudo apt install openvpn
2. Create auth file that contains user name and password
3. Step #2
    ```
    $ cd /etc/openvpn
    $ sudo wget https://www.privateinternetaccess.com/openvpn/openvpn.zip
    $ sudo unzip openvpn.zip
    $ sudo rm openvpn.zip
    ```
 4. Paste contents of config file
 ```sudo vi /etc/openvpn/client.conf```
 5. Create up.sh and down.sh files in openvpn directory
 
     ```
    # Up.sh
    #!/bin/sh
    
    echo "Starting Transmission Torrent Downloading"
    
    transmission-remote --auth xxxx:xxxx --torrent all --start
    ```
    ```
    # down.sh
    #!/bin/sh
    
    echo "Stopping Transmission Torrent Downloading"
    
    transmission-remote --auth xxxx:xxxx --torrent all --stop
    ```

5. Start OpenVPN
    ```
    sudo openvpn --config /etc/openvpn/client.conf --daemon
    ```
6. View logs
    ```
    sudo cat /var/log/openvpn.log
    ```

7. Test to see if VPN is working
    ```
    curl -s checkip.dyndns.org|sed -e 's/.*Current IP Address: //' -e 's/<.*$//'
    ```

8. Enable OpenVPN at Boot
    ```
    sudo vi /etc/default/openvpn
    ```
    Uncomment one of the AUTOSTART lines and set it to this:
    ```
   AUTOSTART=”client”
   ```