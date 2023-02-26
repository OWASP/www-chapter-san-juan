# Building our Attack & Defense H@kb0x

## cywf

We want to do the following, and would appriciate your help setting this up:

- a raspberry pi running Ubuntu Server 20.04 LTS
- need to set it up to be headless, so we don't have to have a monitor to use it
- it'll be accessed via an ubuntu VM
- we need to reformat the drive through the VM
- here are the CURRENT drive details:
    - name: SATAFIRM S11 Media
    - size: 120.03GB
    - ssd: PNY 250GB SATA 2.5 SATA Drive
- here is the DESIRED drive details:
    - partitions = 3 (size accordingly to match desired outcome below)
        - first partition: iPXE (will recieve an image from specified IP: which will be from partition 3 aka our NTFS server that will recieve instructions from our VPS)
        - second partition: Backup / Swap partition
        - third partition: NTFS drive (that will communicate to our VPS and recieve instructions, transfer files, facilitated via our Zerotier admin network)
- we would like to create a cloud-init script that the NTFS partition will serve to the first partition (running iPXE)

How should we go about achieving this? Could you break it down into steps for me, so we can then go step by step?

## ChatGPT

Sure, I'd be happy to help! Here are the steps to achieve what you're looking for:

* **Step 1**: Set up Raspberry Pi with Ubuntu Server 20.04 LTS

To set up the Raspberry Pi with Ubuntu Server 20.04 LTS, you will need to download the operating system image from the Ubuntu website and flash it onto a microSD card using a tool such as Etcher. Insert the microSD card into the Raspberry Pi and power it on.

* **Step 2**: Configure the Raspberry Pi for headless operation

To configure the Raspberry Pi for headless operation, you will need to enable SSH and set a static IP address. You can do this by creating a file called "ssh" in the boot partition of the microSD card, and by modifying the network configuration file located at /etc/netplan/50-cloud-init.yaml to set a static IP address.

* **Step 3**: Access the Raspberry Pi via Ubuntu VM

To access the Raspberry Pi via Ubuntu VM, you will need to ensure that the virtual network adapter is configured to use bridged networking. This will allow the virtual machine to obtain an IP address on the same network as the Raspberry Pi.

* **Step 4**: Reformat the drive through the VM

To reformat the drive through the VM, you can use a tool such as GParted. Install GParted on the Ubuntu VM and connect the Raspberry Pi to the virtual machine using SSH. You can then use GParted to format the drive to your desired partition layout.

* **Step 5**: Create the desired partition layout

To create the desired partition layout, you can use GParted to create three partitions on the drive. The first partition should be formatted as FAT32 and will be used to store the iPXE bootloader. The second partition should be formatted as either swap or ext4 and will be used as a backup partition or as a swap partition. The third partition should be formatted as NTFS and will be used to store files and communicate with the VPS.

* **Step 6**: Create a cloud-init script

To create a cloud-init script, you will need to install cloud-init on the Raspberry Pi. Once installed, you can create a YAML file that specifies the configuration for the NTFS partition, including any packages to install, users to create, and files to download. This file can then be served from the NTFS partition to the first partition running iPXE using a tool such as TFTP.

I hope this helps you get started with your project! Let me know if you have any questions or need further clarification on any of the steps.

## cywf 

Okay, then lets start with Step 1. How can I go about doing this in linux? Could you provide me with the commands to do this? Perhaps in a bash script?

## ChatGPT

Sure! Here are the commands to download and flash the Ubuntu Server 20.04 LTS image onto a microSD card in Linux:

```bash
#!/bin/bash

# Download Ubuntu Server 20.04 LTS image
wget https://cdimage.ubuntu.com/releases/20.04.3/release/ubuntu-20.04.3-live-server-arm64.iso

# Verify checksum
wget https://cdimage.ubuntu.com/releases/20.04.3/release/SHA256SUMS
sha256sum --ignore-missing -c SHA256SUMS 2>&1 | grep OK

# Insert microSD card and identify its device name
lsblk

# Unmount microSD card
sudo umount /dev/sdX1

# Flash image onto microSD card
sudo dd bs=4M if=ubuntu-20.04.3-live-server-arm64.iso of=/dev/sdX conv=fsync status=progress
```

Note that you will need to replace `/dev/sdX` with the **device name** of your **microSD card**, which you can identify using the `lsblk` command. Also, make sure to modify the URLs in the wget commands if you want to download a different version of Ubuntu Server.

## cywf

Okay great! Let's proceed with Step 2: "Configure the Raspberry Pi for headless operation"

## ChatGPT

Sure thing! Here are the commands to enable SSH and set a static IP address on the Raspberry Pi:

```bash
#!/bin/bash

# Mount the boot partition
sudo mount /dev/mmcblk0p1 /mnt

# Enable SSH
sudo touch /mnt/ssh

# Set static IP address
cat << EOF | sudo tee /mnt/network-config
network:
  version: 2
  ethernets:
    eth0:
      dhcp4: no
      addresses:
        - 192.168.1.100/24 
      gateway4: 192.168.1.1
      nameservers:
        addresses: [1.1.1.1, 9.9.9.9]
EOF

# Unmount the boot partition
sudo umount /mnt
```

Note that you will need to modify the IP address, gateway, and nameserver addresses to match your network configuration. Also, make sure to run this script on your Linux machine and not on the Raspberry Pi itself. Once you have run this script and inserted the microSD card into the Raspberry Pi, it should be accessible via SSH using the static IP address you set.

## cywf

Okay, great! Let's proceed with Step 3: "Access the Raspberry Pi via Ubuntu VM"

## ChatGPT

Sure thing! Here are the steps to access the Raspberry Pi via an Ubuntu VM:

1. Make sure your Ubuntu VM is running and connected to the same network as the Raspberry Pi.

2. Determine the IP address of the Raspberry Pi by checking your router's DHCP client list or by using a network scanning tool like nmap.

3. Open a terminal on your Ubuntu VM and use the ssh command to connect to the Raspberry Pi using its IP address. For example: ssh username@192.168.1.100, where username is the username you created when setting up the Raspberry Pi and 192.168.1.100 is the IP address you assigned to the Raspberry Pi in Step 2.
    
4. If prompted, enter the password for the username account on the Raspberry Pi.

Once you have successfully connected to the Raspberry Pi via SSH, you can use it just like you would if you were using it directly with a monitor and keyboard.

## cywf

Okay, great! Let's proceed with Step 4: "Reformat the drive through the VM"

## ChatGPT

Sure thing! Here are the steps to reformat the drive on the Raspberry Pi via the Ubuntu VM:

1. Make sure you are logged in to the Raspberry Pi via SSH from your Ubuntu VM.

2. Check the current partition table of the drive by running the command `sudo parted /dev/sda print`.

3. If there are any existing partitions on the drive, delete them by running the command `sudo parted /dev/sda rm <partition_number>`. Replace `<partition_number>` with the number of the partition you want to delete, as listed in the output of the previous command. Repeat this command for each partition you want to delete.

4. Create the **first partition** for _iPXE_ by running the command `sudo parted /dev/sda mkpart primary 1MiB 100MiB`.

5. Create the **second partition** for _backup/Swap_ by running the command `sudo parted /dev/sda mkpart primary 100MiB 4GiB`.

6. Create the **third partition** for the _NTFS_ drive by running the command `sudo parted /dev/sda mkpart primary 4GiB 100%`.

7. Format the first partition as _FAT32_ by running the command `sudo mkfs.vfat /dev/sda1`.

8. Format the second partition as _ext4_ by running the command `sudo mkfs.ext4 /dev/sda2`.

9. Format the third partition as _NTFS_ by running the command `sudo mkfs.ntfs /dev/sda3`.

10. Label the **first partition** as _"iPXE"_ by running the command `sudo e2label /dev/sda1 iPXE`.

11. Label the **second partition** as _"Backup"_ by running the command sudo e2label /dev/sda2 Backup.

12. Label the **third partition** as _"NTFS"_ by running the command `sudo ntfslabel /dev/sda3 NTFS`.

Note that you will need to replace `/dev/sda` with the **device name** of your **SSD**, which you can _identify_ using the `lsblk` command. Also, make sure to adjust the partition sizes to match the desired sizes you specified in your initial request.

## cywf

Okay, great! Since Step 4 pretty much included Step 5 as well, let's proceed with Step 6: "Create a cloud-init script"

## ChatGPT

Sure thing! Here are the steps to create a cloud-init script:

1. Open a text editor on your Ubuntu VM.

2. Add the following lines to the file to configure the first partition for iPXE:

```bash
#cloud-config
output: {all: '| tee -a /var/log/cloud-init-output.log'}
runcmd:
  - echo 'console=tty1 console=ttyS0,115200n8' | sudo tee -a /boot/cmdline.txt
  - echo 'iPXE boot' | sudo tee /boot/config.txt
  - echo '#!ipxe' | sudo tee /boot/start.elf
```

This script adds the console=tty1 and console=ttyS0,115200n8 options to the kernel command line in /boot/cmdline.txt, sets the text "iPXE boot" in /boot/config.txt, and creates a file /boot/start.elf that contains the iPXE script.

3. Add the following lines to the file to configure the third partition for NTFS:

```bash
#cloud-config
output: {all: '| tee -a /var/log/cloud-init-output.log'}
runcmd:
  - sudo apt-get update
  - sudo apt-get install -y ntfs-3g
  - sudo mkdir /mnt/ntfs
  - echo "/dev/sda3 /mnt/ntfs ntfs-3g defaults,auto,uid=1000,gid=1000 0 0" | sudo tee -a /etc/fstab
```

This script updates the package list, installs the ntfs-3g package to enable read/write access to NTFS partitions, creates a mount point at /mnt/ntfs, and adds an entry to /etc/fstab to automatically mount the NTFS partition at boot time.

4. Save the file with a descriptive name, such as ipxe-and-ntfs.yaml.

5. Copy the file to the first partition of the Raspberry Pi by running the command sudo cp ipxe-and-ntfs.yaml /mnt/iPXE/user-data.

6. Unmount the first partition by running the command sudo umount /mnt/iPXE.

7. Safely eject the SD card from your Ubuntu VM.

Once you have completed these steps, you should be able to insert the SD card into the Raspberry Pi and power it on. The cloud-init script will be executed automatically during the first boot, configuring the first and third partitions as specified.

## cywf

Okay great! Lets review the steps we have completed so far, and what (if anything) is left to do

## ChatGPT

Sure thing! Here is a summary of the steps we have completed so far:

1. We have installed Ubuntu Server 20.04 LTS on the Raspberry Pi.

2. We have configured the Raspberry Pi for headless operation.

3. We have accessed the Raspberry Pi via an Ubuntu VM.

4. We have reformatted the drive through the VM, creating three partitions: one for iPXE, one for backup/swap, and one for NTFS.

5. We have created a cloud-init script that configures the first partition for iPXE and the third partition for NTFS.

At this point, it should be possible to insert the SD card into the Raspberry Pi and power it on. The cloud-init script will be executed automatically during the first boot, configuring the partitions as specified.

However, depending on your specific use case, there may be additional configuration or software installation steps required. For example, you may need to configure your network settings, install additional packages, or configure firewall rules. If you have any additional requirements, feel free to let me know and I can provide further guidance

## cywf

Nice, I think thats all for today, though when we return we will continue building the device! 