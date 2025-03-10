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
* run `iwctl` to open a wireless connectivity tool (the prompt will change to `[iwd]#`)
* run `station <name_of_your_wireless_interface> get-networks` to get the name of all the available networks on that interface
* run `station <name_of_your_wireless_interface> connect <name_of_your_network>` and enter the password once prompted
* run `exit` to exit the `iwctl` utility
* `ip addr show` to check if there is an IP address now :)
* `ping archlinux.org` to check if we have connectivity

### Notes:
* Typically, wired interfaces are named `eth0`, `enpXsY`, or similar, while wireless interfaces are usually named `wlan0, wlpXsY`, or similar. 

## Partitioning our Disk
* Make sure you check out this [video](https://www.youtube.com/watch?v=2Z6ouBYfZr8) if you want to understand storage volumes and formatting in depth :)
* We need to know the device identifier for the hard drive where we want to install Arch
* run `lsblk` to show all the storage volumes on the computer and note the name of the drive (common ones are nvmeX and sdX)
* You should be able to tell which one is the computer hard drive by its size :)
* run `fdisk /dev/<drive_name> to partition the drive (do not pick a partition like `sda1` or `nvme0n1p1`, you need the whole disk, like `sda` or `nvme0n1`)
* The prompt should change to `Command (m for help)`
* type `p` to check the current partition layout
* type `g` to create a new partiton table (GPT)
* type `p` again and no partitions will be shown
* type `n` to create a new partition 
* type `Enter` to accept the default for the partition number
* type `Enter` to accept the default first sector
* type `+1G` to make its size 1G (it might ask you to override a previous signature, press `Y` to accept)
* type `n` to create another partition
* type `Enter` to accept the default
* type `Enter` again to accept the default
* type `+1G` to make its size 1G as well and remove the previous signature if prompted
* type `p` to check what we have so far
* type `n` to create a new partition 
* type `Enter` to accept the default for the partition number
* type `Enter` to accept the default first sector
* type `Enter` to accept the default last sector which will use up all of our hard drive space
* type `t` to change the type of a partition and select `3` 
* type `L` to list all the types and note the number that corresponds to Linux LVM (in my case 44)
* press `q` to exit the list and type the number that you noted (44)
* type `p` to double check the partition layout
* No changes have been written to the disk yet, you just created a layout for now :)
* type `w` to write the changes to the disk. **There is no going back from this point on!!!!**

### Notes:
* Why even partition the disk?
    * Organization: Separates system files from user data.
    * Security: Isolates critical files from user data.
    * Backup and Recovery: Simplifies backing up specific data.
    * Multiple Operating Systems: Allows installation of multiple OSes.

* What is a partition table?
    * A partition table is a data structure on a storage device (like a hard drive or SSD) that defines how the drive is divided into partitions. It contains information about:
        * Partition Size: The size of each partition.
        * Partition Type: The file system type or purpose (e.g., primary, extended, logical).
        * Starting and Ending Addresses: The locations on the disk where each partition begins and ends.
        * Boot Information: Details about which partition is bootable.
    * Common partition table formats include Master Boot Record (MBR) and GUID Partition Table (GPT). The partition table is essential for the operating system to manage and access the partitions on the drive.

## Formatting our Partitions
* Format the partitions to use FAT or ext4 filesystems
* run `mkfs.fat -F32 /dev/<partition_1>` (this could be something like `sda1` or `nvme0n1p1`)
* The command above makes the first partition a FAT filesystem
* Make the second partition an ext4 filesystem
* run `mkfs.ext4 /dev/<partition_2>` 
* For now let's encrypt the 3rd partition, we'll format it later :)
* run `cryptsetup luksFormat /dev/<partiton_3>` to encrypt the third partition using LUKS 
* type `YES` to confirm LUKS encryption setup and create a passphrase
* That passphrase will be used everytime you boot your computer, so don't forget it :)
* Now the partition is encrypted but first we need to open it to be able to work with it
* run `cryptsetup open --type luks /dev/<partition_3> <name>` (<name> gives the partition a name, you can use `lvm`, because we are going to format it to use Linux LVM)
* type the password to unlock the partition

### Notes:
* Formatting a partition with mkfs.fat or mkfs.ext4 is essential for:
    * Creating a File System: Enables the OS to manage files.
    * Initializing Data Structures: Sets up necessary components for the file system.
    * Clearing Previous Data: Removes existing data for a fresh start.
    * Defining Storage Management: Establishes how data is stored and accessed.

* FAT (File Allocation Table):
    * Simple file system for flash drives and memory cards.
    * Widely compatible with many devices.
    * Limited file size (e.g., FAT32 has a 4GB limit).

* ext4 (Fourth Extended File System):
    * Advanced file system used in Linux.
    * Supports large files and partitions.
    * Includes journaling for better data safety and performance.

* LUKS (Linux Unified Key Setup) is a standard for disk encryption in Linux. It provides:
    * Full Disk Encryption: Secures entire partitions or drives to protect data.
    * Key Management: Allows multiple user keys for accessing encrypted data.
    * Compatibility: Works with various file systems and is widely supported in Linux distributions.
    * Security: Uses strong encryption algorithms to safeguard data from unauthorized access.
* LUKS is commonly used to enhance data security on laptops and servers.

## Configuring LVM
* Let's configure LVM on the third partition
* Make sure you check out the [LVM video](https://www.youtube.com/watch?v=MeltFN-bXrQ) if you want to understand LVM in depth
* We need to create a physical volume first
* `pvcreate /dev/mapper/lvm` (/dev/mapper/lvm is the name of the encrypted third drive)
* `vgcreate volgroup0 /dev/mapper/lvm ` to create a volume group and name it `volgroup0`
* `lvcreate -L 30GB volgroup0 -n lv_root` to create a logical volume within `volgroup0` with size `30G` and name `lv_root`
* Think of a logical volume as partition in the usual sense, that's what we're going to be formatting and using :)
* `lvcreate -l 100%FREE -volgroup0 -n lv_home` to allocate the remaining disk space to logical volume `lv_home`
* `vgdisplay` to show all the volume groups
* `lvdisplay` to show all logical volumes, which should have `lv_root` and `lv_home`
* `modprobe dm_mod` to load the Device Mapper kernel module to Linux, don't worry about it too much, it makes LVM possible :)
* `vgscan` to scan for volume groups
* `vgchange -ay` to activate all volume groups
* Now we can finish formatting the logical volumes as ext4
* `mkfs.ext4 /dev/volgroup0/lv_root` and `mkfs.ext4 /dev/volgroup0/lv_home`
* `mount /dev/volgroup0/lv_root /mnt` to mount the volume at /mnt
* This makes the file system on that logical volume accessible at the /mnt directory.
* `mkdir /mnt/boot` to create a boot directory to mount the boot partition
* `mount /dev/<partition_2> /mnt/boot` to mount the partition at /mnt/boot
* Don't worry about mounting the first partition for now, we still have some work to do there :)
* `mkidr /mnt/home` and `mount /dev/volgroup0/lv_home /mnt/home` to mount home volume at /mnt/home

### Notes:

* LVM (Logical Volume Manager) is a system for managing disk storage in Linux. It provides:
    * Flexible Storage Management: Allows you to create, resize, and delete logical volumes easily.
    * Dynamic Allocation: Enables the allocation of disk space on-the-fly without needing to repartition.
    * Snapshots: Supports creating snapshots of volumes for backups or recovery.
    * Volume Grouping: Combines multiple physical disks into a single logical volume group for easier management.
* LVM enhances the flexibility and efficiency of managing disk space in Linux systems.

## Installing required packages
* Now we can go on to install the system and all the required packages
* `pacstap -i /mnt base` to install the base Arch system into the /mnt directory :)

## Generate the fstab file
* `genfstab -U -p /mnt >> /mnt/etc/fstab` generates a file system table (fstab) for the newly installed Linux system
* This is required for mounting partitions at boot time :)
* `cat /mnt/etc/fstab` You should see three partitions listed in the file 

## Using arch-chroot to finish the installation
* Log into the in-progress Arch system to make configure some more things :)
* `arch-chroot /mnt` will do just that and the prompt will change to `[root@archiso /]#`

### Notes:
* `arch-chroot` is used to change the root directory to a specified location (like the new system's root) during the Arch Linux installation. Why Use chroot?
    * Configuration: It allows you to configure the new system as if you are running it, making it easier to set up.
    * Isolation: Changes made in the chroot environment only affect the new installation, not the host system.
    * Package Installation: You can install packages and modify system files directly in the new environment.
* In short, `arch-chroot` helps you set up and manage the new Arch Linux system effectively.

## Setting up Users
* `passwd` to set a password for the root account
* `useradd -m -g users -G wheel <username>` to add a user and include it in the `wheel` group
* `passwd <user>` to change the password for the user account :)

### Notes:
* The `wheel` group is used to manage and restrict administrative access, allowing only designated users to perform tasks that require elevated privileges.

* `wheel` Group:
    * Purpose: Traditionally used to grant users administrative access.
    * Usage: In some systems, only users in the `wheel` group can use the `sudo` command to perform tasks as the root user.
    * History: It comes from older Unix systems and is still used in some Linux distributions (like Arch Linux).

* `sudo` Group:
    * Purpose: A more modern way to grant users administrative access.
    * Usage: In many distributions (like Ubuntu), users in the `sudo` group can use the `sudo` command to run commands with elevated privileges.
    * Simplicity: It simplifies management of who can perform administrative tasks.

* Why Both?
    * Flexibility: Some systems use the `wheel` group, while others use the `sudo` group. It depends on the distribution and how it’s set up    
    * Compatibility: The wheel group is still around for compatibility with older systems.

## Install Basic Packages
* `pacman -S base-devel dosfstools grub efibootmgr lvm2 mtools nano networkmanager openssh` will install most of the packages you need
* `pacman` is the installation tool for Arch Linux and other Arch-based distributions :)

## Install the Linux kernel and firmware
* `pacman -S linux linux-headers` to install the linux kernel 
* `pacman -S linux-firmware` to install the firware

### Notes:
* The kernel is the fundamental part of an operating system that enables it to function by managing hardware and system resources, making it essential for the overall operation of the OS.
* Firmware is a type of software that is embedded into hardware devices to control and manage their functions. 
* `linux-firmware` provides additional firmware files that the Linux kernel can load when needed.

## Install GPU files
* `lspci` to list the PCI devices
* If you have an Intel or AMD GPU use `pacman -S mesa`
* If you have an Nvidia GPU use `pacman -S nvidia nvidia-utils`
* If you need support for accelerated video decoding, consider installing these packages:
    * Intel (Broadwell and newer): `intel-media-driver`
    * Intel GMA 4500 up to Coffee Lake: `libva-mesa-driver`
    * AMD: `libva-mesa-driver`
    * Nvidia: Already taken care of with `nvida-utils`

### Notes:
* PCI (Peripheral Component Interconnect) devices are hardware components that connect to a computer's motherboard via the PCI bus. This bus allows for communication between the motherboard and various peripheral devices. Examples of PCI devices:
    * Graphics cards (GPUs)
    * Network interface cards (NICs)
    * Sound cards
    * Storage controllers (e.g., SATA, SCSI)
    * USB expansion cards
    * Modems

* Hardware Decoding is the use of dedicated hardware to decode video data, providing benefits such as improved performance, power efficiency, and support for high-resolution video playback. It is widely used in devices like TVs, smartphones, and computers to enhance the viewing experience.

## Edit mkinitcpio.conf
* `vim /etc/mkinitcpio.conf` to configure how the initramfs is built. (You can also use `nano` to edit files)
* Go to the line that is uncommented and starts with `HOOKS` 
* Add `encrypt lvm2` in between `block` and `filesystems`

### Notes:
* `mkinitcpio.conf` file is used to configure the `mkinitcpio` tool, which is responsible for `initramfs` 
* `initramfs` stands for initial RAM filesystem. It is a temporary filesystem that is loaded into memory during the boot process of a Linux system. 
* The primary purpose of `initramfs` is to provide a minimal environment that allows the Linux kernel to initialize the system and mount the root filesystem.
* The `HOOKS` line in `mkinitcpio.conf` specifies the order of hooks which are modules and scripts that will be included in the initramfs. 
* The hooks determine what actions are taken during the boot process.

## Generate kernel ramdisks
* `mkinitcpio -p linux` to generate `initramfs`
* Edit `/etc/locale.gen` file and uncomment the line `en_US.UTF-8` (delete the `#` symbol at the beginning of the line)
* `locale-gen` to generate the locale
* Locales define language and regional settings, such as date formats, currency, and character encoding

## Set up GRUB
* `vim /etc/default/grub` to edit the grub config file :)
*  Look for a line containing `GRUB_CMDLINE_LINUX_DEFAULT` and add `cryptdevice=/dev/<partition_3>:<volume_group_name>` to the end of it. So, something like this `cryptdevice=/dev/nvme0n1p3:volgroup0` 
* `mkdir /boot/EFI` to create the `EFI` directory 
* `mount /dev/<partition_1> /boot/EFI` to mount the EFI partition at `/boot/EFI` 
* Install `GRUB` using `grub-install --target=x86_64-efi --bootloader-id=grub_uefi --recheck`
* `cp /usr/share/locale/en\@quot/LC_MESSAGES/grub.mo /boot/grub/locale/en.mo` to copy locale files for GRUB’s messaging to our boot directory
* `grub-mkconfig -o /boot/grub/grub.cfg` to generate a config file for GRUB :)
* `systemctl enable NetworkManager` to enable NetworkManager so we can set up connectivity after boot (kinda important lol) :)

### Notes:
* GRUB, which stands for Grand Unified Bootloader, is a program that helps your computer start up. When you turn on your computer, GRUB is the first thing that runs. It allows you to choose which operating system to boot if you have more than one installed. GRUB also loads the necessary files to start the operating system you select. 
* The EFI (Extensible Firmware Interface) partition is a small section of a hard drive that is used in systems with UEFI (Unified Extensible Firmware Interface) firmware. It stores bootloaders for operating systems, allowing the system to start up properly. 

## The FINISH Line :)
* `exit` to exit the chroot environment
* `umount -a` to unmount all partitions 
* REBOOT :) (lol the command is `reboot`)

Please check out all the videos from the following [playlist](https://www.youtube.com/watch?v=RcFlZjpvf0U&list=PLT98CRl2KxKHKd_tH3ssq0HPrThx2hESW). Learn Linux TV is an absolutely amazing channel that helped me start my Linux journey :)














