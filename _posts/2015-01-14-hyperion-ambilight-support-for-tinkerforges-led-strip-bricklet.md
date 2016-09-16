---
id: 238
title: 'Hyperion (Ambilight) Support for Tinkerforge's LED Strip Bricklet'
date: 2015-01-14T21:21:25+00:00
author: Björn Bilger
layout: post
guid: https://bbilger.com/yabapal/?p=238
permalink: /2015/01/14/hyperion-ambilight-support-for-tinkerforges-led-strip-bricklet/
categories:
  - Linux
  - Raspberry Pi
---
Wow, it's been a while since my last post. I totally forgot to mention that I added support for <a title="Tinkerforge" href="http://tinkerforge.com" target="_blank">Tinkerforge</a>&#8216;s <a title="LED Strip Bricklet" href="http://tinkerforge.com/doc/Hardware/Bricklets/LED_Strip.html" target="_blank">LED Strip Bricklet</a> to <a title="Hyperion" href="https://github.com/tvdzwan/hyperion" target="_blank">Hyperion</a> &#8211; about 10 months, ago.

For those of you who don't know Hyperion, yet: It&#8217;s an &#8216;Ambilight&#8217; implementation that in contrast to the original <a title="boblight" href="https://code.google.com/p/boblight/" target="_blank">boblight</a> runs quite fast on the Raspberry Pi (RPi). It supports <a title="Raspbmc" href="http://raspbmc.com/" target="_blank">Raspbmc</a>, <a title="XBian" href="http://xbian.org/" target="_blank">XBian</a> and <a title="OpenELEC" href="http://openelec.tv/" target="_blank">OpenELEC</a> but it is not limited to those Linux distributions specialized on letting XBMC/Kodi run on the RPi.

After you installed Hyperion, as described <a title="Hyperion Installation Guide" href="https://github.com/tvdzwan/hyperion/wiki/installation" target="_blank">here</a>, you need a configuration. You can create the configuration file with a tool called <a title="HyperionCon" href="https://raw.github.com/tvdzwan/hypercon/master/deploy/HyperCon.jar" target="_blank">HyperionCon</a> (more details can be found <a title="Hyperion Configuration" href="https://github.com/tvdzwan/hyperion/wiki/configuration" target="_blank">here</a>). This tool, however, does not support the Tinkerforge device. This means you need to edit the &#8220;device&#8221; section of your hyperion.config.json manually.

<!--more-->

<pre class="brush: jscript; title: ; notranslate" title="">{
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
</pre>

  * name(string): leave it as it is
  * type(string): make sure to use &#8220;tinkerforge&#8221; //mandatory
  * output(string): the ip of the device where you connected your Master Brick to/where <a title="brick daemon" href="http://www.tinkerforge.com/doc/Software/Brickd.html" target="_blank">brick daemon</a> is running at //optional; default: 127.0.0.1
  * port(int): the port your <a title="brick daemon" href="http://www.tinkerforge.com/doc/Software/Brickd.html" target="_blank">brick daemon</a> is running at //optional; default: 4223
  * uid(string): the UID of your LED Strip Bricklet //mandatory; you can find it out via Brick Viewer
  * colorOrder(string): you can change the colorOrder if you have problems //optional; default rgb
  * rate(int): the frame duration //mandatory; <frame duration>= 1 / FPS; note there's some bug => make sure to set this value!!!

Note 1: at the moment only &#8220;WS2801&#8221; is supported

Note 2: Unfortunately my pull request has been merged into Hyperion without  a dependency on <https://github.com/bbilger/libtinkerforge>. If you are interested into the one with that dependency, then check out: [https://github.com/bbilger/add_tinkerforge](https://github.com/bbilger/add_tinkerforge "https://github.com/bbilger/add_tinkerforge")

Note 3: I also forgot to release support for LCDProc. I will do so, soon.