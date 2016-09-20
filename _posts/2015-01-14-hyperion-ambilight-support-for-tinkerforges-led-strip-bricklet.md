---
id: 238
title: 'Hyperion (Ambilight) Support for Tinkerforge''s LED Strip Bricklet'
date: 2015-01-14T21:21:25+00:00
author: Björn Bilger
permalink: /2015/01/14/hyperion-ambilight-support-for-tinkerforges-led-strip-bricklet/
categories:
  - Linux
  - Raspberry Pi
---
Wow, it's been a while since my last post. I totally forgot to mention that I added support for [Tinkerforge](Tinkerforge" href="http://tinkerforge.com)'s [LED Strip Bricklet](http://tinkerforge.com/doc/Hardware/Bricklets/LED_Strip.html) to [Hyperion](https://github.com/tvdzwan/hyperion) - about 10 months, ago.

For those of you who don't know Hyperion, yet: It's an 'Ambilight' implementation that in contrast to the original [boblight](https://code.google.com/p/boblight/) runs quite fast on the Raspberry Pi (RPi). It supports [Raspbmc](http://raspbmc.com/), [XBian](http://xbian.org/) and [OpenELEC](http://openelec.tv/) but it is not limited to those Linux distributions specialized on letting XBMC/Kodi run on the RPi.

After you installed Hyperion, as described [here](https://github.com/tvdzwan/hyperion/wiki/installation), you need a configuration. You can create the configuration file with a tool called [HyperionCon](https://raw.github.com/tvdzwan/hypercon/master/deploy/HyperCon.jar) (more details can be found [Hyperion Configuration](https://github.com/tvdzwan/hyperion/wiki/configuration)). This tool, however, does not support the Tinkerforge device. This means you need to edit the **device** section of your hyperion.config.json manually.

<!--more-->

``` json
{
...
        "device" :
        {
                "name"       : "MyPi",
                "type"       : "tinkerforge",
                "output"     : "127.0.0.1",
                "port"       : 4223,
                "uid"        : "xyz",
                "colorOrder" : "rbg",
                "rate" : 50
        },
...
}
```

  * name(string): leave it as it is
  * type(string): make sure to use **tinkerforge** //mandatory
  * output(string): the IP of the device where you connected your Master Brick to/where [brick daemon](http://www.tinkerforge.com/doc/Software/Brickd.html) is running at //optional; default: 127.0.0.1
  * port(int): the port your [brick daemon](http://www.tinkerforge.com/doc/Software/Brickd.html) is running at //optional; default: 4223
  * uid(string): the UID of your LED Strip Bricklet //mandatory; you can find it out via Brick Viewer
  * colorOrder(string): you can change the colorOrder if you have problems //optional; default rgb
  * rate(int): the frame duration //mandatory; <frame duration>= 1 / FPS; note there's some bug => make sure to set this value!!!

Note 1: at the moment only **WS2801** is supported

Note 2: Unfortunately my pull request has been merged into Hyperion without a dependency on <https://github.com/bbilger/libtinkerforge>. If you are interested into the one with that dependency, then check out: <https://github.com/bbilger/add_tinkerforge>

Note 3: I also forgot to release support for LCDProc. I will do so, soon.
