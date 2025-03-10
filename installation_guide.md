# Arch Installation Guide
Many thanks to Learn Linux TV

## Downloading the ISO image
* Go to the [download page](https://archlinux.org/download/) and proceed to one of the HTTP/HTTPS mirrors. 
* Download the file that ends with `.iso` (it should be the first file on the list)

## Create bootable installation media
* Here is a [detailed guide](https://www.youtube.com/watch?v=0xuP1GQLPpI) on how to accomplish that :)
* You can also create a [Ventoy](https://www.youtube.com/watch?v=10L8aCY3VBs) bootable flash drive, which allows you to have multiple `ISO` files in a single drive :)

### Notes:
* Make sure to check out the [Arch Linux Wiki page](https://wiki.archlinux.org/title/Main_page), it's an amazing resource for all Arch Linux and other Linux related learning material :)

## Boot into the Installation media
* This is different on every computer, you need to go to the [UEFI settings](https://www.windowscentral.com/software-apps/windows-11/how-to-enter-uefi-on-devices-running-windows-11) and make sure that you boot into the flash drive on which you installed the `ISO` file
* If [Secure Boot](https://support.microsoft.com/en-us/windows/windows-11-and-secure-boot-a8ff1202-c0d9-42f5-940f-843abef64fad) is enabled you have to disable it for the duration of the installation, but you can enable it after the system is up and running :)

## Connectig to the Internet
* If you are using Ethernet you should already be connected to the Internet
* Use the command `ip addr show` to show interfaces 
* If it shows an IP iddress on either the wired or wireless interface you are connected to the Internet and should be good to go :)
* Here is how to connect to Wi-Fi:
* `ip addr show` and note the name of the wireless interface
* run `iwctl` to open a wireless connectivity tool (the prompt will change)
* run `station <name_of_your_wireless_interface> get-networks` to get the name of all the available networks on that interface
* run `station <name_of_your_wireless_interface> connect <name_of_your_network>` and enter the password once prompted
* run `exit` to exit the `iwctl` utility
* `ip addr show` to check if there is an IP address now :)
* `ping archlinux.org` to check if we have connectivity

### Notes:
* Typically, wired interfaces are named `eth0`, `enpXsY`, or similar, while wireless interfaces are usually named `wlan0, wlpXsY`, or similar. 

## Partitioning our Disk
* We need to know the device identifier for the hard drive where we want to install Arch
* run `lsblk` to show all the storage volumes on the computer and note the name of the drive (common ones are nvmeX and sdX)
* You should be able to tell which one is the computer hard drive by its size :)
* run `fdisk /dev/<drive_name> to partition the drive 

### Notes:
* Why even partition the disk?
    * Organization: Separates system files from user data.
    * Security: Isolates critical files from user data.
    * Backup and Recovery: Simplifies backing up specific data.
    * Multiple Operating Systems: Allows installation of multiple OSes.














