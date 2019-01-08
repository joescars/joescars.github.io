---
id: 79
title: Installing Windows 10 IoT on Raspberry Pi 2 from Windows 8
date: 2015-07-03T12:35:25+00:00
author: Joe
layout: post
guid: http://joeraio.com/?p=79
permalink: /installing-windows-10-iot-on-raspberry-pi-2-from-windows-8/
image: /assets/wp-content/uploads/2015/07/windows-10-iot-raspberry-pi-ii.jpg
categories:
  - IoT
  - Raspberry Pi
  - Windows 10
tags: [iot, internet of things, raspberry pi, windows 10]
---
![Installing Windows 10 IoT on Raspberry Pi 2 from Windows 8](/assets/wp-content/uploads/2015/07/windows-10-iot-raspberry-pi-ii.jpg)

So you have a Raspberry Pi 2 and want to install Windows 10 IoT on it but don’t have a machine running Windows 10? No problem. These instructions will guide you through how to accomplish this.

It is important to note that this is not the “official” way to install it. These instructions allow you to do the install from a machine running Windows 7 or Windows 8/8.1.

With that said continue at your own risk! 🙂

#### Tools I am using today 

  * Raspberry Pi2
  * Microsoft Surface Pro 3
  * Sandisk 64gb Class 10 MicroSD Card
  * Targus USB SD Card Reader 
      * Even though the Surface Pro 3 has a MicroSD slot, I had to use the external reader for the software used in the install to recognize the card properly

#### Installation Instructions

  * Login to  <http://connect.microsoft.com> using your Microsoft Account
  * Go to Directory -> Windows Embedded -> Join &#8220;_Windows Developer for IoT_&#8221; program

![Windows 10 IoT Program](/assets/wp-content/uploads/2015/07/microsoft-connect-win-10-iot-program.png)

  * Once joined, go to downloads and download **Windows 10 IoT Core Insider Preview Image for Raspberry Pi 2**
  * At the time of this post the latest build is 5/12/2015
  * From here you can download the zip directly or use the [Microsoft File Transfer Manager](http://transfers.ds.microsoft.com/ftm/default.aspx?target=install)

![Windows 10 IoT Core Download](/assets/wp-content/uploads/2015/07/windows-iot-core-download.png)

  * We now need to download three more utilities so that we can do the install from Windows 8 or Windows 7 
      * [SD Formatter](https://www.sdcard.org/downloads/formatter_4/index.html) – I always format my SD card before starting
      * [ImgMount](http://forum.xda-developers.com/attachment.php?attachmentid=1593476&d=1356534867) – This lets you mount the image and create a VHD from the mounted image.
      * [WinImage](http://www.winimage.com/download.htm) – This program lets you write the VHD to an eternal drive
  * I took everything and copied to one folder on my machine to make things easier. In the example I named it c:\w10iot

![Windows 10 IoT Folder](/assets/wp-content/uploads/2015/07/w10iot-folder.png)

  * Extract the Windows 10 IoT zip file once downloaded 
      * The only file we are going to work with today is **Flash.ffu**
  * Launch an administrative command prompt and browse to c:\w10iot
  * Run the following command “imgmount flash.ffu”

![Windows 10 IoT Create VHD](/assets/wp-content/uploads/2015/07/windows-10-iot-create-vhd.png)

  * Windows+X and go to "Computer Management"
      1. Go to Disk Management
          1. You should see an additional drive, right click on it and “detach it”
          2. Make sure to note the folder where the VHD is located
              1. I copied and pasted this into a notepad
          3. Move the file into your c:\w10iot folder

![Detach VHD](/assets/wp-content/uploads/2015/07/windows-10-iot-detach-vhd.png)

![VHD Location](/assets/wp-content/uploads/2015/07/windows-10-iot-vhd-location.png)

  * Install WinImage
      * No license code is needed. It comes with a free 30 day trial. Of course if you like the software I encourage you to purchase it!
  * Plug your MicroSD card in
      * Again; I do this using the reader
  * Install SD Formatter and format your card
      * I use the following options
          1. Format Type – FULL (Erase)
          2. Format Size Adjustment &#8211; OFF

![IoT Format Options](/assets/wp-content/uploads/2015/07/windows-10-iot-sd-format-options.png)

  * Launch WinImage (administrator)
  * Go to Disk -> Restore Virtual Hard Disk image on Physical Drive

![Image Export](/assets/wp-content/uploads/2015/07/win-image-export.png)

  * Choose your MicroSD card drive

![Select Disk](/assets/wp-content/uploads/2015/07/win-image-select-disk.png)

  * Select your VHD

![Select VHD](/assets/wp-content/uploads/2015/07/win-image-select-vhd.png)

  * It will warn you and then start the process
  * Depending on your hardware it should take about 5-10 minutes

![Image in Progress](/assets/wp-content/uploads/2015/07/win-image-progress.png)

  * Once its done place the MicroSD card in your Raspberry Pi2 and boot it up!
  * The install will take a while.

![Windowns 10 IoT Setup](/assets/wp-content/uploads/2015/07/windows-10-iot-setup.jpg) 
![Windows 10 IoT First Boot](/assets/wp-content/uploads/2015/07/windows-10-iot-first-boot.jpg)

  * That’s it! Happy developing!

![Windows 10 IoT Complete](/assets/wp-content/uploads/2015/07/windows-10-iot-setup-complete.jpg)