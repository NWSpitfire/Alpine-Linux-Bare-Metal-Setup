# Alpine Linux Bare-Metal Setup
 Setup of Alpine linux on bare metal systems

###### Some of this guide uses extracts from other online guides. Please see the credits section at the bottom for credits.


# Prerequesits

# Installation
## Preperation

1. Load Alpine to USB

## System Install

1. Set Boot Partition to 500MB.

        export BOOT_SIZE=500

2. Set system Swap sizing. 

        export SWAP_SIZE=2048

    ###### NOTE: On storage-limited systems such as Thin Clients, less should be used. [Current Setup on a Wyse 3030LT with 4GB eMMC is 1024MB swap].

3. Set system Bootloader type.

        export BOOTLOADER=grub

4. Identify the system's internal drive alias in preperation for system setup.

        df -h

###### NOTE: On most systems it is likely to be /dev/sda1. However on some systems (like thin clients) it may be different due to them using different storage technologies. For example, Thin client might me /dev/mmcblk0.

5. Run Alpine system setup script (Installation).

        setup-alpine

This script will run through and setup Alpine's configuration. The parameters to be setup are:

    1. Keyboard and Variant.

    2. Hostname.

    3. Network options: 
    
    3a. Select the eth0 adapter, this is the onboard ethernet port)

    3b. You can either specify DHCP or setup static networking.

    3c. DNS Options. It is recommended to use 8.8.8.8 and none for the domain

    4. Time zone options: Just use the suggested defaults.

    5. Proxy Options: Use noneif you are connecting directly to the Internet.

    6. SSH Options. Select OpenSSH.

    7. NTP Options. Chrony is a good choice.

    8. Boot Mode: 
    
    8a. Select sys to install the system on disk.

    8b. TBA.

    9. Disk Options. Select the system disk based on drive identified in Step 4. Most systems will use **sda**, however some (Thin Clients) might use *mmcblk0**.

This setup will create the following disk structure.

    - **/dev/sda1** as BOOT (500MB) in **/boot**
    - **/dev/sda2** as SWAP (2GB (if 2048 specified), or 1GB (if 1024 specified))
    - **/dev/sda3** as ROOT (rest of available disk space).

9. After these have run through the system will be fully initialised and Alpine will be installed to the internal system disk. Reboot to apply.

        reboot

# System Familiarisation

This section is a WIP, but is used to highlight functions of Alpine and how they contrast to Debian (to make conversion to this OS easier).

## Basic Syntax

 - apk = Alpine Package Keeper. The debian alternative to this would be apt. This is used to manage packages in the Alpine OS, such as upgrade or install packages.
 - add = Install a Package, or upgrade an existing one.
 - del = Delete an installed package.

### Basic Syntax Examples

- Update the system package list.

        apk update

- Install pending system updates.

        apk upgrade

- Install a package.

        apk add <package name>

### Become the root user

        su

### Exit the root user terminal

        su -



# Install Packages

This section is optional but used to install other applications such as docker or Wireless drivers.

**Before installing any services, its bet to run the commands below as a root user. To enter the root terminal, run;**

    su

## Fixing the package manager.

On initial install, Alpine is configured to use the **MAIN** package repository to fetch its packages. This is fine, but some packages, such as Docker, are only available on the **COMMUNITY** package repository. By default, the COMMUNITY repository is disabled, meaning if you try to install a package (in this case Docker), which is only available on the COMMUNITY repository, then you will get an error;

    ERROR: unable to select packages:
    docker (no such package):
    required by: world[docker]

To fix this, you will have to enable the **COMMUNITY** repository. 

1. Check the installed repository lists.

        tail -f /etc/apk/repositories

You should see both the **/main** and **/community** are listed, but the **/community** is commented out with a **#**.

###### NOTE: Press "CTRL + C" to exit.

2. Run Vim editor to enable the community repository.

        vi /etc/apk/repositories

3. Remove the **#** in front of the **/community** link.

4. To save the file, press the escape key. Then key the following;

        :wq

###### BREAKDOWN: w = Write file, q = quit.

5. Update the package list with the new repository.

        apk update

You should see the **/community** version be printed. Now you can go ahead and install packages.

## Installing Packages

To search for available packages and their names, use the links below.

[Official Package Index](https://pkgs.alpinelinux.org/packages)

[Linux Package Index](https://pkgs.org/)
###### NOTE: This package index features ALL linux distros, to search for Alpine specifically, you have to use the filters.

** Before starting, don't forget to become the root user.**

    su

### Install Nano file editor

    apk add nano



### Install Docker & Docker Compose

Installing Docker is a several step process.

1. Download and install the package.

        apk add docker

2. Connect to Docker by adding yourself to the docker group

        addgroup ${USER} docker

3. Set Docker to start at system boot, and run now.

        rc-update add docker default
        service docker start

4. Now that Docker is installed and setup, Docker Compose needs to be installed.

        apk add docker-cli-compose

#### Test to make sure docker works correctly.

    docker run --name docker-nginx -p 80:80 nginx

Go to the systems IP address, on port 80 and you should see the Nginx landing page!.

###### Press "CTRL + C" to exit.

![Nginx](https://github.com/NWSpitfire/Alpine-Linux-Bare-Metal-Setup/blob/main/Pictures/nginx.png)


        
### Install htop to view system usage in a more human readable format

    apk add htop

###### NOTE: "top" is installed by default, I just prefer htop.



## Wi-Fi Setup

This section is specifically for setting up the Realtek WiFi dongle on my system.

**Run commands as root user**

    su

-----

1. Update the system package repository.

        apk update

2. Install WPA Supplicant.

        apk add wpa_supplicant

3. Install the Realtek Wi-Fi driver.

        apk add --upgrade linux-firmware-rtlwifi

[Package info and supported devices](https://alpine.pkgs.org/3.20/alpine-main-x86_64/linux-firmware-rtlwifi-20240811-r0.apk.html)

4. List network adapters. Look for the wlan0 specifially to make sure it is listed.

        ip a

    ###### NOTE: If you don't see wlan0, try rebooting. If not see the Troubleshooting section.

5. Add your wireless credentials to the WPA Supplicant configuration file.

        wpa_passphrase 'ExampleWifiSSID' 'ExampleWifiPassword' > /etc/wpa_supplicant/wpa_supplicant.conf

    ###### NOTE: The small quotation mark encasing the SSID and Password are required.

    ![Start WPA](https://github.com/NWSpitfire/Alpine-Linux-Bare-Metal-Setup/blob/main/Pictures/WPA-config-file.png)

6. Start WPA Supplicant in the foreground to confirm it works.

        wpa_supplicant -i wlan0 -c /etc/wpa_supplicant/wpa_supplicant.conf

    ###### If this prints successful connection (as pictured below), press "CTRL + C" to stop it and get back to the terminal.

    ![Check WPA](https://github.com/NWSpitfire/Alpine-Linux-Bare-Metal-Setup/blob/main/Pictures/WPA-Connection-Test.png)

7. If Step 6 worked, run it in the background.

        wpa_supplicant -B -i wlan0 -c /etc/wpa_supplicant/wpa_supplicant.conf

    ![background WPA](https://github.com/NWSpitfire/Alpine-Linux-Bare-Metal-Setup/blob/main/Pictures/WPA-Success.png)

8. Check the adapter has come up (it won't have an IP yet).

        ip a


9. Configure the interface with an IP address.

        udhcpc -i wlan0

    ![DHCP](https://github.com/NWSpitfire/Alpine-Linux-Bare-Metal-Setup/blob/main/Pictures/DHCP.png)

10. Check the interface has successfully leased an IP from the DHCP Server.

        ip addr show wlan0

    ![DHCP Success](https://github.com/NWSpitfire/Alpine-Linux-Bare-Metal-Setup/blob/main/Pictures/DHCP%20Success.png)


11. Set the Wi-Fi adapter and DHCP to start automatically on boot.

        nano /etc/network/interfaces

12. Add wlan0 definition to the file

        auto wlan0
        iface wlan0 inet dhcp

    ![WLAN Auto](https://github.com/NWSpitfire/Alpine-Linux-Bare-Metal-Setup/blob/main/Pictures/wlan-auto.png)

13. Test to make sure everything is configured correctly and the interface will come up as expected. First bring the interface down.

        ip link set wlan0 down

14. Now manually restart the networking service.

        rc-service networking --quiet restart &

    ![WLAN test](https://github.com/NWSpitfire/Alpine-Linux-Bare-Metal-Setup/blob/main/Pictures/wlan-test.png)

15. Now that everything is setup and works, set WPA supplicant to start automatically on boot.

        rc-update add wpa_supplicant boot

16. Ensure networking is set to start automatically on boot.

        rc-update add networking boot

    ![WLAN boot](https://github.com/NWSpitfire/Alpine-Linux-Bare-Metal-Setup/blob/main/Pictures/wlan-boot.png)    

### Setting DHCP to update automatically.

With the above configuration, udhcpc will only run once at boot. If the Wifi isn't available then, or the network changes after booting, udhcpc needs to be notified. You can automatically notify udhcpc of network changes by using a wpa_cli action file, such as the one installed by default at `/etc/wpa_supplicant/wpa_cli.sh`.

- **MANUALLY**

        wpa_cli -a /etc/wpa_supplicant/wpa_cli.sh

- **AUTOMATICALLY**

1. Open the WPA CLI file.

        nano /etc/conf.d/wpa_cli

2. Copy the update command into the file.

        WPACLI_OPTS="-a /etc/wpa_supplicant/wpa_cli.sh"

    ###### NOTE: If it is already present, this step is not needed.

3. Set the WPA CLI to start on system boot.

        rc-update add wpa_cli boot


# Credits and Links

[Initial Setup Guide](https://venenux.github.io/alpine-wiki/tutorials/alpine-install-from-usb-to-disk-pc-single-boot-only.html)

[Alpine Package Index](https://pkgs.alpinelinux.org/packages)

[Linux RTLWiFi Firmware](https://alpine.pkgs.org/3.20/alpine-main-x86_64/linux-firmware-rtlwifi-20240811-r0.apk.html)

[Fixing Alpine Package Error](https://www.hasanaltin.com/error-unable-to-select-packages-error-on-alpine-linux/)

[Alpine Docker](https://wiki.alpinelinux.org/wiki/Docker)

[Alpine Wi-Fi](https://wiki.alpinelinux.org/wiki/Wi-Fi#Troubleshooting)
