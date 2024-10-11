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


### Install Nano file editor.

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
        
### Install htop to view system usage in a more human readable format

        apk add htop

###### NOTE: "top" is installed by default, I just prefer htop.

### 



# Credits and Links

[Initial Setup Guide](https://venenux.github.io/alpine-wiki/tutorials/alpine-install-from-usb-to-disk-pc-single-boot-only.html)

[Alpine Package Index](https://pkgs.alpinelinux.org/packages)

[Linux RTLWiFi Firmware](https://alpine.pkgs.org/3.20/alpine-main-x86_64/linux-firmware-rtlwifi-20240811-r0.apk.html)

[Fixing Alpine Package Error](https://www.hasanaltin.com/error-unable-to-select-packages-error-on-alpine-linux/)

[Alpine Docker](https://wiki.alpinelinux.org/wiki/Docker)
