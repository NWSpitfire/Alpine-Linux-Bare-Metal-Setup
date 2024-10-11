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

###### NOTE: On most systems it is likely to be /dev/sda1. However on some systems (like thin clients) it may be different due to them using different storage technologies. For example, Thin client might me mmcblk0.

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

# System Initialisation

# Credits and Links

[Initial Setup Guide](https://venenux.github.io/alpine-wiki/tutorials/alpine-install-from-usb-to-disk-pc-single-boot-only.html)

[Alpine Package Index](https://pkgs.alpinelinux.org/packages)

[Linux RTLWiFi Firmware](https://alpine.pkgs.org/3.20/alpine-main-x86_64/linux-firmware-rtlwifi-20240811-r0.apk.html)
