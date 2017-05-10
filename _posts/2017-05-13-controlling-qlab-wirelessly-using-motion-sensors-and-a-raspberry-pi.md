---
layout: post
title: Controlling QLab Wirelessly Using Motion Sensors and a Raspberry Pi
published: true
---

## Overview 
This year, we are setting up Halloween scenes that are triggered by motion.  As someone passes by the scene, the scene is triggered.  We are using [QLab](https://figure53.com/qlab/) to control our scenes.  QLab is show control software for the Mac.

I wanted the ability to trigger QLab's playback, from motion sensors, wirelessly.  QLab supports trigger notifications via [OSC](https://en.wikipedia.org/wiki/Open_Sound_Control) (Open Sound Control) network commands.  

[I-CubeX](http://infusionsystems.com/) recently released the [PiShield](https://infusionsystems.com/pishield/) for the Raspberry Pi that supports interfacing sensors to the Raspberry Pi.  The Raspberry Pi supports [Node-Red](https://nodered.org/), which is a programming tool for wiring together hardware devices, APIs and online services.  Node-Red will be able to convert sensor data to OSC messages.  The Raspberry Pi Zero W supports WIFI and will be able to transmit the sensor data wirelessly to QLab.

I primarily wanted to keep track of the steps I took to set this up.  This article is work in progress.

## Components
The following components are required for this setup.
 - [Raspberry Pi Zero W](https://www.raspberrypi.org/products/pi-zero-w/)
 - [Raspberry Pi Power Supply](https://www.raspberrypi.org/products/universal-power-supply/)
 - 8GB Micro SD Card
 - Micro SD Card Reader/Writer
 - [I-CubeX PiShield](https://infusionsystems.com/pishield/)
 - [Board-To-Board Connector](http://www.newark.com/amp-te-connectivity/2-826925-0/connector-header-40-position-2row/dp/12H4415)
 - [Motion Sensors](http://infusionsystems.com/catalog/product_info.php/products_id/413) - Up to 8 are supported
 - [QLab](https://figure53.com/qlab/)
 - Wireless WIFI Network

![Components](/images/components.jpg)

## Hardware Setup

Setup the Raspberry Pi and the PiShield following the directions [here](https://infusionsystems.com/pishield/documentation/hardware-setup/).  In the case of the Pi Zero, the male header will need to be soldered to the board.  The final result should look as follows:

![Hardware Setup](/images/1.png)

## Headless Raspberry Pi Setup
I opted for setting up the Raspberry Pi without using a keyboard or monitor attached to the Raspberry Pi.  This simplifies the overall setup as an extra keyboard and monitor is not needed.  I referenced [this](https://hackernoon.com/raspberry-pi-headless-install-462ccabd75d0) blog post for initial information.

### Step 1: Download Raspbian
You can download Raspbian image from [here](https://www.raspberrypi.org/downloads/raspbian/).  Download Raspbian Jessie with Pixel.  The Lite version may work, but I have not tried it.  At the time of this writing, I downloaded the April 2017 version.

![Download Raspbian](/images/download_raspbian.png)

Unzip the downloaded file.  You should have an .iso file similar to the name *2017-04-10-raspbian-jessie.img*

### Step 2: Write ISO Image to SD Card
The Raspbian image needs to be written to the SD card so the Raspberry Pi can boot off the image.  [Win32 Disk Imager](https://sourceforge.net/projects/win32diskimager/) can be used to write the ISO image to an SD card on Windows.  [Etcher](https://etcher.io/) can also be used which supports Mac, Linux, and Windows.  Instructions are [here](https://www.raspberrypi.org/documentation/installation/installing-images/README.md).  

![Write ISO Image to SD Card](/images/write-image-to-sd-card.png)

### Step 3: Enable SSH and Configure WIFI
In order to initially allow SSH access to the Raspberry Pi, an empty **ssh** file needs to be added to the root of the boot partition on the SD card.

To support WIFI, create a **wpa_supplicant.conf** file, that contains your WiFi settings.  This file also needs to be added to the root of the boot partition on the SD card.  The contents of the file should be as follows:

```
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
network={
    ssid="SSID"
    psk="PASSWORD"
}
```

![Enable SSH and Configure WIFI](/images/enable-ssh-and-configure-wifi.png)

### Step 4: Boot the Raspberry Pi
Insert the SD you prepared into the Raspberry Pi.  Power on the Raspberry Pi by plugging in the power.

### Step 5: Determine the IP address of the Raspberry Pi
You can use an app such as [Fing](https://www.fing.io/) to discover the IP address of your Raspberry Pi.  Alternatively, you can look at what IP address your router assigned to the Raspberry Pi.  Remember this IP address as you will need it later.

![Determine the IP address of the Raspberry Pi](/images/determine-ip-address-of-the-raspberry-pi.png)

## Configure the Raspberry Pi
We need to configure and install the appropriate software onto the Raspberry Pi for the setup to work.

### Step 1: SSH into the Raspberry Pi
Because we are using a Headless setup, we will setup and configure the Raspberry Pi over SSH.  [Putty](http://www.putty.org/) is the Windows client I typically use for SSH sessions.  Connect to the Raspberry Pi using the IP address previously discovered.  

![SSH into the Raspberry Pi](/images/ssh-into-the-raspberry-pi.png)

The default credentials are:
```
login as: pi
password: raspberry
```

![SSH into the Raspberry Pi](/images/ssh-into-the-raspberry-pi-user-pass.png)

### Step 2: Basic Configuration
Set the clock.  The current date must not be too many days off, otherwise the NTPD service will not set the actual date.  Set the date to the current date.
```
sudo date -s "2017-05-10"
```


Configure the basic settings of the Raspberry Pi.  Run the following command:
```
sudo raspi-config
```

![Basic Configuration](/images/basic-configuration.png)

Using the Raspberry Pi Software Configuration Tool, configure the following:

1. Boot Options > Desktop / CLI > Console
1. Boot Options > Wait for Network at Boot > Yes
1. Localisation Options > Change Locale > [en_US.UTF-8 UTF-8]
1. Localisation Options > Change Timezone > [US > Eastern]
1. Interfacing Options > SPI / YES
1. Interfacing Options > IC2 / YES
1. Select Finish / Yes to Reboot

Reconnect to the Raspberry Pi after reboot via SSH.

### Step 3: Update and Upgrade Raspbian
Make sure Raspbian is up to date.  Run the following commands:
```
sudo apt-get update
sudo apt-get upgrade
```
### Step 4: Install Node-Red
[Node-Red](https://nodered.org/docs/hardware/raspberrypi) comes pre-installed on Raspbian with Pixel.  We need to update Node-Red and install required add-ons.
 
To upgrade Node-Red, run the following command.  This may take a while.

```
update-nodejs-and-nodered
```

Create the .node-red folder to hold the add-ons
```
cd ~
mkdir .node-red
```
Install Node-Red mcp3008 support.  [Reference](https://infusionsystems.com/pishield/node-red-sensor-acquisition/)
```
cd ~/.node-red
npm install mcp3008
```
Install Node-Red OSC support. [Reference](https://www.npmjs.com/package/node-red-contrib-osc)
```
cd ~/.node-red
npm install node-red-contrib-osc
```
Set Node-Red to Auto Start
```
cd ~
sudo systemctl enable nodered.service
```

Start Node-Red in background (Node-Red will start automatically on boot.)
```
node-red-start&
```

## Setup Node-Red Flow

TODO
