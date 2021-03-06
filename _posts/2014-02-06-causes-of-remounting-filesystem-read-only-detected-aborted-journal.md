---
id: 219
title: 'Causes of "Remounting filesystem read-only" / "Detected aborted journal"'
date: 2014-02-06T00:57:42+00:00
author: Björn Bilger
permalink: /2014/02/06/causes-of-remounting-filesystem-read-only-detected-aborted-journal/
categories:
  - Raspberry Pi
---
I just bought another RPi (for some home automation project) and had some tough time setting it up. The issue was that it constantly died with some error which resulted in **Detected aborted journal** or rather **Remounting filesystem read-only** making it necessary to restart and fix the filesystem with fcsk. It was simply impossible to install new packages or update the firmware.

I did some research and the suggested solutions were to buy a better power supply but the supplied power was ok - well, finally I bought a new one, but it still did not work. I tweaked some settings in config.txt but again it did not work.

So now my setup comes into play: I had some USB hub with a USB flash drive connected to it (OS boots from the flash drive). The problem now was that the driver and/or the hub's chipset was simply corrupt in the sense of being unable to cope with flash drives or rather high-speed devices (no complains about connecting low-speed devices like keyboards).

When I connected the flash drive to the RPi directly it worked without any problems and it was lots faster.

So now the question remains: why didn't I checked this earlier? Last time (for my other RPi) I had the same issue and there the solution was to buy a USB hub and let it power the flash drive instead of the RPi.

The USB hub was an Digitus DA-70231  with the hardware id 05e3:0608 (Genesys Logic, Inc. USB-2.0 4-Port HUB). This chipset is marked as working and not working here: [http://elinux.org/RPi\_Powered\_USB_Hubs](http://elinux.org/RPi_Powered_USB_Hubs). So from my experience it is working for low-speed devices but not for high-speed devices.
