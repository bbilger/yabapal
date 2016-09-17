---
id: 165
title: 'Styling Fedora''s Xfce spin like Xubuntu'
date: 2013-07-07T22:03:26+00:00
author: Björn Bilger
layout: post
guid: https://bbilger.com/yabapal/?p=165
permalink: /2013/07/07/styling-fedora-like-xubuntu/
categories:
  - Linux
tags:
  - Fedora
  - style
  - theme
  - Xfce
  - Xubuntu
---
My start into Linux/Unix began with Ubuntu and Gnome (2!). Since I neither liked Gnome 3 nor Unity and didn't like the idea of having yet another fork with Mate or Cinnamon, I migrated to Xfce and so to Xubuntu a couple of years ago. After the (recent) controversial debates about Mir, Unity, the dispute with Debian about patches and so Canoncial's attitude toward the Open Source community in general, I decided to switch to another distro. Regarding the fact that my server is running on CentOS I decided to give Fedora a try on my desktop machine.

Leaving aside all other aspects/differences for now, I was kind of shocked by the design of Fedora with Xfce. The reason is that Fedora's Xfce spin is using a vanilla version of Xfce. This vanilla version, even though arguable, is no delight to behold for me. I almost switched back to Xubuntu but decided to change the style instead and give it a fair try. So this guide will summarize the steps to give Fedora's Xfce spin Xubuntu's style and make it a delight to behold, again.

There are a couple of guides/posts out there about exactly this but most seem to be outdated. Many of those posts are talking about downloading the packages from the ubuntu repository, downloading the themes, etc. This is, however, not necessary anymore. The themes are already installed and you just need to install a few packages via yum and change some settings. It's fairly simple - at least nowadays with Fedora 18/19.

So this is what I did:

<!--more-->

First thing you want to do is to open the application menu's properties menu and uncheck _Show button title_. This will remove the unnecessary text on that button. In addition you can change the icon to the Fedora's, if you like.

Next you need to install two packages:

``` bash
sudo yum install google-droid* elementary-icon-theme
```

The first one will give you the **Droid Sans** font and the other will give you another icon set. Both of these are used by Xubuntu.

The last step is changing some settings.

Navigate to Settings->Settings Manager->Appearance.

In the style tab change the style to **Greybird**.

In the icons tab change the icon to **elementary Dark**.

In the fonts tab, change the font to **Droid Sans** with size 10, check **Enable anti-aliasing**, change the hinting value to **Slight** (or whatever fits your needs), the sub-pixel order to **RGB** (or whatever fits your need), check **Custom DPI setting** and set its value to 96.

In the settings tab you need to check **Show images buttons** and **Show images menus**.

Navigate to Settings->Settings Manager->Window Manager.

In the style tab you need to change the theme to **Greybird** and set the title font to **Droid Sans Bold** with size 9.

Basically that's it. Next you can adjust the panel's to your needs and/or adjust whatever themes and settings you like.

If you still have the feeling that your fonts do not look proper in your browser and other applications, you can install [Infinality](http://www.infinality.net/):

``` bash
sudo rpm -Uvh http://www.infinality.net/fedora/linux/infinality-repo-1.0-1.noarch.rpm
sudo yum install freetype-infinality fontconfig-infinality
```

Note: Currently there's no repo for Fedora 19. In order to install, you have to temporarily replace **$releaseserver** by **18** in `/etc/yum.repos.d/infinality.repo`.

Even if you are fine with your current font rendering, it's really worth giving [Infinality](http://www.infinality.net/) a try! Fonts are amazingly crisp; it's even better than on Ubuntu by default and probably better or at least as good as on Windows.

Hope this helps you to not run away from Fedora with Xfce immediately 😉
