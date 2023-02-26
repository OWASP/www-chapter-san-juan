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

