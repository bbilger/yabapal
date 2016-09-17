---
id: 223
title: libtinkerforge
date: 2014-03-03T22:18:36+00:00
author: Björn Bilger
layout: post
guid: https://bbilger.com/yabapal/?p=223
permalink: /2014/03/03/libtinkerforge/
categories:
  - Linux
tags:
  - Linux
  - Tinkerforge
---
I recently purchased some [Tinkerforge](http://www.tinkerforge.com/) bricklets and plan to extend some programs ([Hyperion](https://github.com/tvdzwan/hyperion), [Boblight](http://code.google.com/p/boblight/) and [LCDproc](http://lcdproc.omnipotent.net/)) to add support for the devices or rather bricklets ([LED Strip](http://www.tinkerforge.com/en/doc/Hardware/Bricklets/LED_Strip.html) and [LCD](http://www.tinkerforge.com/en/doc/Hardware/Bricklets/LCD_20x4.html)). For that reason I want or rather need to have a Tinkerforge library but unfortunately the guys at Tinkerforge think of it a little different:

> We do not offer a pre-compiled library, since it would be a pain in the ass to provide them for all combinations of architectures and operating systems. <http://www.tinkerforge.com/en/doc/Software/API_Bindings_C.html>

Well, I think it is a pain in the ass not to offer such a library - not even a Makefile to create one.

So I created one: <https://github.com/bbilger/libtinkerforge>

The current release version is 1.0.0 and you can download it from GitHub, as well: <https://github.com/bbilger/libtinkerforge/releases/download/v1.0.0/tinkerforge-1.0.0.tar.gz>

Input is welcome!

Have fun!
