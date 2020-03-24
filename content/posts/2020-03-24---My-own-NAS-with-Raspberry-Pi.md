---
title: My own NAS with Raspberry Pi
date: "2020-03-24T22:12:03.284Z"
template: "post"
draft: false
slug: "/posts/My-own-NAS-with-Raspberry-Pi"
category: "Homelab"
tags:
  - "NAS"
  - "Raspberry-Pi"
  - "OpenMediaVault"
  - "smb"
description: "How I set up my own NAS with a Raspberry Pi 4"
---

I have always wanted my own NAS seeing that I have limited storage on my laptop. Since I have an external hard drive, and a raspberry pi just laying around, I decided to finally do it.
I have tha following goals for this project, and in this post I'm going to start from the first.

1. Set up OpenMediaVault and enable smb
2. Use the Raspberry Pi NAS as my time machine drive

## What I used

### Hardware

1. Raspberry Pi 4
2. SanDisk Extreme 128 GB micro SD card
3. Raspberry Pi Universal Power Supply
4. Seagate Backup Plus Portable 4TB

### Software

1. OpenMediaVault 5
2. Raspbian
3. balenaEtcher

## Steps

### 1. Installing Raspbian

#### Preparing the micro SD card

First Download Raspbian [here](https://www.raspberrypi.org/downloads/raspbian/)

Then use [balenaEtcher](https://www.balena.io/etcher/) to flash the image onto the SD card.

Because I am going to use ssh to connect to the Raspberry Pi, I need to enable SSH by first creating an emty `ssh` file at the root of the file system by running:

```bash
touch /Volumes/boot/ssh
```

After the steps above are done I can eject the micro SD card from my computer and insert it into the Raspberry Pi.

Now with the micro SD card prepared, I can power up the RPI with it connected to the internet via a ethernet cable.

#### Preparing Raspbian

To connect to the RPI, I need its ip address, which I obtained using nmap as below. Remember to change the command to match your network setting.

```bash
nmap -sP 192.168.2.0/24
```

There should be output something like below:

```text
Nmap scan report for raspberrypi (192.168.2.195)
Host is up (0.00027s latency).
MAC Address: XX:XX:XX:XX:XX:XX (Raspberry Pi Trading)
```

Now I know to connect to the RPI at 192.168.2.195 (refered onward as RPI_IP). So I logged in with the default user **pi** and password **raspberry**.

```bash
ssh pi@<RPI_IP>
```

Remember to change the password once logged in by typing `passwd`.

To make sure the **pi** user can log in more than once via ssh, add **pi** into the **ssh** group.

```bash
sudo adduser pi ssh
```

Finnaly make sure that Raspbian is up to date.

```bash
sudo apt-get update
sudo apt-get upgrade
```

### 2. Installing OMV 5

#### Installing OMV

Installing OMV on a Raspberry Pi is extremely easy, simply run a single line.

```bash
wget -O - https://github.com/OpenMediaVault-Plugin-Developers/installScript/raw/master/install | sudo bash
```

After the script finishes, reboot the RPI.

```bash
sudo reboot
```

Once booted, I could see the GUI by entering RPI_IP in the browser address bar. The default user is **admin** with the password **openmediavault**.

![OMV UI image](/media/2020-03-24---My-own-NAS-with-Raspberry-Pi/omv-ui-01.png)

#### Configuring OMV

Go to System > General Settings > Interfaces to add an ethernet interface.

![OMV add internet interface](/media/2020-03-24---My-own-NAS-with-Raspberry-Pi/omv-ui-02.png)

Select the wired Ethernet interface and select DHCP.

![OMV add internet interface](/media/2020-03-24---My-own-NAS-with-Raspberry-Pi/omv-ui-03.png)

I also added a new user and enabled home directories in Access Rights Management > User.

### 3. Mount the external hard drive

Make sure the disk is connected in Storage > Disks

![OMV check disks](/media/2020-03-24---My-own-NAS-with-Raspberry-Pi/omv-ui-04.png)

Then go to Storage > File Systems to create a new file system for that drive

![OMV create fs](/media/2020-03-24---My-own-NAS-with-Raspberry-Pi/omv-ui-05.png)

Remember to mount the file system.

![OMV mount fs](/media/2020-03-24---My-own-NAS-with-Raspberry-Pi/omv-ui-06.png)

Now I can see the usage on the file system.

![OMV view fs](/media/2020-03-24---My-own-NAS-with-Raspberry-Pi/omv-ui-07.png)

### 4. Enable SMB

I went to Services > SMB/CIFS to enable SMB. I enabled Home directories so that each user can have their own folder.

Now I can see my user folder (rootn) when I connect to the SMB server.

![OMV view fs](/media/2020-03-24---My-own-NAS-with-Raspberry-Pi/finder-omv.png)

## Reference

[OpenMediaVault Document](https://github.com/OpenMediaVault-Plugin-Developers/docs/blob/master/Adden-B-Installing_OMV5_on_an%20R-PI.pdf)
