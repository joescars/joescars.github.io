---
id: 79
title: Installing Windows 10 IoT on Raspberry Pi 2 from Windows 8
date: 2015-07-03T12:35:25+00:00
author: Joe
layout: post
guid: http://joeraio.com/?p=79
permalink: /installing-windows-10-iot-on-raspberry-pi-2-from-windows-8/
price_table:
  - 'a:0:{}'
image: /assets/wp-content/uploads/2015/07/windows-10-iot-raspberry-pi-ii.jpg
categories:
  - IoT
  - Raspberry Pi
  - Windows 10
---
So you have a Raspberry Pi 2 and want to install Windows 10 IoT on it but don’t have a machine running Windows 10? No problem. These instructions will guide you through how to accomplish this.

It is important to note that this is not the “official” way to install it. These instructions allow you to do the install from a machine running Windows 7 or Windows 8/8.1.

With that said continue at your own risk! 🙂

**Tools I am using today **

  * Raspberry Pi2
  * Microsoft Surface Pro 3
  * Sandisk 64gb Class 10 MicroSD Card
  * Targus USB SD Card Reader 
      * Even though the Surface Pro 3 has a MicroSD slot, I had to use the external reader for the software used in the install to recognize the card properly

**Installation Instructions**

  * Login to  <http://connect.microsoft.com> using your Microsoft Account
  * Go to Directory -> Windows Embedded -> Join &#8220;_Windows Developer for IoT_&#8221; program

[<img class="alignnone size-full wp-image-83" src="https://joeraio.com/wp-content/uploads/2015/07/microsoft-connect-win-10-iot-program.png" alt="microsoft-connect-win-10-iot-program" width="544" height="246" srcset="http://localhost/wp-content/uploads/2015/07/microsoft-connect-win-10-iot-program.png 544w, http://localhost/wp-content/uploads/2015/07/microsoft-connect-win-10-iot-program-300x136.png 300w" sizes="(max-width: 544px) 100vw, 544px" />](https://joeraio.com/wp-content/uploads/2015/07/microsoft-connect-win-10-iot-program.png)

  * Once joined, go to downloads and download **Windows 10 IoT Core Insider Preview Image for Raspberry Pi 2**
  * At the time of this post the latest build is 5/12/2015
  * From here you can download the zip directly or use the [Microsoft File Transfer Manager](http://transfers.ds.microsoft.com/ftm/default.aspx?target=install)

[<img class="alignnone size-full wp-image-85" src="https://joeraio.com/wp-content/uploads/2015/07/windows-iot-core-download.png" alt="windows-iot-core-download" width="992" height="293" srcset="http://localhost/wp-content/uploads/2015/07/windows-iot-core-download.png 992w, http://localhost/wp-content/uploads/2015/07/windows-iot-core-download-300x89.png 300w" sizes="(max-width: 992px) 100vw, 992px" />](https://joeraio.com/wp-content/uploads/2015/07/windows-iot-core-download.png)

  * We now need to download three more utilities so that we can do the install from Windows 8 or Windows 7 
      * [SD Formatter](https://www.sdcard.org/downloads/formatter_4/index.html) – I always format my SD card before starting
      * [ImgMount](http://forum.xda-developers.com/attachment.php?attachmentid=1593476&d=1356534867) – This lets you mount the image and create a VHD from the mounted image.
      * [WinImage](http://www.winimage.com/download.htm) – This program lets you write the VHD to an eternal drive
  * I took everything and copied to one folder on my machine to make things easier. In the example I named it c:\w10iot

[<img class="alignnone size-full wp-image-87" src="https://joeraio.com/wp-content/uploads/2015/07/w10iot-folder.png" alt="w10iot-folder" width="258" height="190" />](https://joeraio.com/wp-content/uploads/2015/07/w10iot-folder.png)

  * Extract the Windows 10 IoT zip file once downloaded 
      * The only file we are going to work with today is **Flash.ffu**
  * Launch an administrative command prompt and browse to c:\w10iot
  * Run the following command “imgmount flash.ffu”

[<img class="alignnone size-full wp-image-88" src="https://joeraio.com/wp-content/uploads/2015/07/windows-10-iot-create-vhd.png" alt="windows-10-iot-create-vhd" width="497" height="241" srcset="http://localhost/wp-content/uploads/2015/07/windows-10-iot-create-vhd.png 497w, http://localhost/wp-content/uploads/2015/07/windows-10-iot-create-vhd-300x145.png 300w" sizes="(max-width: 497px) 100vw, 497px" />](https://joeraio.com/wp-content/uploads/2015/07/windows-10-iot-create-vhd.png)

  * Windows+X and go to “Computer Management” 
      1. Go to Disk Management 
          1. You should see an additional drive, right click on it and “detach it”
          2. Make sure to note the folder where the VHD is located 
              1. I copied and pasted this into a notepad
          3. Move the file into your c:\w10iot folder

[<img class="alignnone size-full wp-image-89" src="https://joeraio.com/wp-content/uploads/2015/07/windows-10-iot-detach-vhd.png" alt="windows-10-iot-detach-vhd" width="555" height="434" srcset="http://localhost/wp-content/uploads/2015/07/windows-10-iot-detach-vhd.png 555w, http://localhost/wp-content/uploads/2015/07/windows-10-iot-detach-vhd-300x235.png 300w" sizes="(max-width: 555px) 100vw, 555px" />](https://joeraio.com/wp-content/uploads/2015/07/windows-10-iot-detach-vhd.png)

[<img class="alignnone size-full wp-image-90" src="https://joeraio.com/wp-content/uploads/2015/07/windows-10-iot-vhd-location.png" alt="windows-10-iot-vhd-location" width="423" height="174" srcset="http://localhost/wp-content/uploads/2015/07/windows-10-iot-vhd-location.png 423w, http://localhost/wp-content/uploads/2015/07/windows-10-iot-vhd-location-300x123.png 300w" sizes="(max-width: 423px) 100vw, 423px" />](https://joeraio.com/wp-content/uploads/2015/07/windows-10-iot-vhd-location.png)

  * Install WinImage 
      * No license code is needed. It comes with a free 30 day trial. Of course if you like the software I encourage you to purchase it!
  * Plug your MicroSD card in 
      * Again; I do this using the reader
  * Install SD Formatter and format your card 
      * I use the following options 
          1. Format Type – FULL (Erase)
          2. Format Size Adjustment &#8211; OFF

[<img class="alignnone size-full wp-image-92" src="https://joeraio.com/wp-content/uploads/2015/07/windows-10-iot-sd-format-options.png" alt="windows-10-iot-sd-format-options" width="428" height="324" srcset="http://localhost/wp-content/uploads/2015/07/windows-10-iot-sd-format-options.png 428w, http://localhost/wp-content/uploads/2015/07/windows-10-iot-sd-format-options-300x227.png 300w" sizes="(max-width: 428px) 100vw, 428px" />](https://joeraio.com/wp-content/uploads/2015/07/windows-10-iot-sd-format-options.png)

  * Launch WinImage (administrator)
  * Go to Disk -> Restore Virtual Hard Disk image on Physical Drive

[<img class="alignnone size-full wp-image-93" src="https://joeraio.com/wp-content/uploads/2015/07/win-image-export.png" alt="win-image-export" width="424" height="276" srcset="http://localhost/wp-content/uploads/2015/07/win-image-export.png 424w, http://localhost/wp-content/uploads/2015/07/win-image-export-300x195.png 300w" sizes="(max-width: 424px) 100vw, 424px" />](https://joeraio.com/wp-content/uploads/2015/07/win-image-export.png)

  * Choose your MicroSD card drive

[<img class="alignnone size-full wp-image-94" src="https://joeraio.com/wp-content/uploads/2015/07/win-image-select-disk.png" alt="win-image-select-disk" width="430" height="310" srcset="http://localhost/wp-content/uploads/2015/07/win-image-select-disk.png 430w, http://localhost/wp-content/uploads/2015/07/win-image-select-disk-300x216.png 300w" sizes="(max-width: 430px) 100vw, 430px" />](https://joeraio.com/wp-content/uploads/2015/07/win-image-select-disk.png)

  * Select your VHD

[<img class="alignnone size-full wp-image-96" src="https://joeraio.com/wp-content/uploads/2015/07/win-image-select-vhd.png" alt="win-image-select-vhd" width="235" height="147" />](https://joeraio.com/wp-content/uploads/2015/07/win-image-select-vhd.png)

  * It will warn you and then start the process
  * Depending on your hardware it should take about 5-10 minutes

[<img class="alignnone size-full wp-image-95" src="https://joeraio.com/wp-content/uploads/2015/07/win-image-progress.png" alt="win-image-progress" width="240" height="189" />](https://joeraio.com/wp-content/uploads/2015/07/win-image-progress.png)

  * Once its done place the MicroSD card in your Raspberry Pi2 and boot it up!
  * The install will take a while.

[<img class="alignnone size-large wp-image-97" src="https://joeraio.com/wp-content/uploads/2015/07/windows-10-iot-setup.jpg" alt="windows-10-iot-setup" width="600" height="352" srcset="http://localhost/wp-content/uploads/2015/07/windows-10-iot-setup.jpg 600w, http://localhost/wp-content/uploads/2015/07/windows-10-iot-setup-300x176.jpg 300w" sizes="(max-width: 600px) 100vw, 600px" />](https://joeraio.com/wp-content/uploads/2015/07/windows-10-iot-setup.jpg) [<img class="alignnone size-large wp-image-98" src="https://joeraio.com/wp-content/uploads/2015/07/windows-10-iot-first-boot.jpg" alt="windows-10-iot-first-boot" width="600" height="361" srcset="http://localhost/wp-content/uploads/2015/07/windows-10-iot-first-boot.jpg 600w, http://localhost/wp-content/uploads/2015/07/windows-10-iot-first-boot-300x181.jpg 300w" sizes="(max-width: 600px) 100vw, 600px" />](https://joeraio.com/wp-content/uploads/2015/07/windows-10-iot-first-boot.jpg)

  * That’s it! Happy developing!

[<img class="alignnone size-full wp-image-99" src="https://joeraio.com/wp-content/uploads/2015/07/windows-10-iot-setup-complete.jpg" alt="windows-10-iot-setup-complete" width="600" height="362" srcset="http://localhost/wp-content/uploads/2015/07/windows-10-iot-setup-complete.jpg 600w, http://localhost/wp-content/uploads/2015/07/windows-10-iot-setup-complete-300x181.jpg 300w" sizes="(max-width: 600px) 100vw, 600px" />](https://joeraio.com/wp-content/uploads/2015/07/windows-10-iot-setup-complete.jpg)