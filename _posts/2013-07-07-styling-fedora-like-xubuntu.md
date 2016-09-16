---
id: 165
title: 'Styling Fedora's Xfce spin like Xubuntu'
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
My start into Linux/Unix began with Ubuntu and Gnome (2!). Since I neither liked Gnome 3 nor Unity and didn't like the idea of having yet another fork with Mate or Cinnamon, I migrated to Xfce and so to Xubuntu a couple of years ago. After the (recent) controversial debates about Mir, Unity, the dispute with Debian about patches and so Canoncial&#8217;s attitude toward the Open Source community in general, I decided to switch to another distro. Regarding the fact that my server is running on CentOS I decided to give Fedora a try on my desktop machine.

Leaving aside all other aspects/differences for now, I was kind of shocked by the design of Fedora with Xfce. The reason is that Fedora's Xfce spin is using a vanilla version of Xfce. This vanilla version, even though arguable, is no delight to behold for me. I almost switched back to Xubuntu but decided to change the style instead and give it a fair try. So this guide will summarize the steps to give Fedora&#8217;s Xfce spin Xubuntu&#8217;s style and make it a delight to behold, again.

There are a couple of guides/posts out there about exactly this but most seem to be outdated. Many of those posts are talking about downloading the packages from the ubuntu repository, downloading the themes, etc. This is, however, not necessary anymore. The themes are already installed and you just need to install a few packages via yum and change some settings. It's fairly simple &#8211; at least nowadays with Fedora 18/19.

So this is what I did:

<!--more-->

First thing you want to do is to open the application menu's properties menu and uncheck &#8220;Show button title&#8221;. This will remove the unnecessary text on that button. In addition you can change the icon to the Fedora&#8217;s, if you like.

Next you need to install two packages:

<pre class="brush: plain; title: ; notranslate" title="">sudo yum install google-droid* elementary-icon-theme
</pre>

The first one will give you the &#8220;Droid Sans&#8221; font and the other will give you another icon set. Both of these are used by Xubuntu.

The last step is changing some settings.

Navigate to Settings->Settings Manager->Appearance.
  
In the style tab change the style to &#8220;Greybird&#8221;.
  
In the icons tab change the icon to &#8220;elementary Dark&#8221;.
  
In the fonts tab, change the font to &#8220;Droid Sans&#8221; with size 10, check &#8220;Enable anti-aliasing&#8221;, change the hinting value to &#8220;Slight&#8221; (or whatever fits your needs), the sub-pixel order to &#8220;RGB&#8221; (or whatever fits your need), check &#8220;Custom DPI setting&#8221; and set its value to 96.
  
In the settings tab you need to check &#8220;Show images buttons&#8221; and &#8220;Show images menus&#8221;.

Navigate to Settings->Settings Manager->Window Manager.
  
In the style tab you need to change the theme to &#8220;Greybird&#8221; and set the title font to &#8220;Droid Sans Bold&#8221; with size 9.

Basically that's it. Next you can adjust the panel&#8217;s to your needs and/or adjust whatever themes and settings you like.

If you still have the feeling that your fonts do not look proper in your browser and other applications, you can install [Infinality](http://www.infinality.net/):

<pre class="brush: bash; title: ; notranslate" title="">sudo rpm -Uvh http://www.infinality.net/fedora/linux/infinality-repo-1.0-1.noarch.rpm
sudo yum install freetype-infinality fontconfig-infinality
</pre>

Note: Currently there's no repo for Fedora 19. In order to install, you have to temporarily replace &#8220;$releaseserver&#8221; by &#8220;18&#8221; in &#8220;/etc/yum.repos.d/infinality.repo&#8221;.

Even if you are fine with your current font rendering, it's really worth giving [Infinality](http://www.infinality.net/) a try! Fonts are amazingly crisp; it&#8217;s even better than on Ubuntu by default and probably better or at least as good as on Windows.

Hope this helps you to not run away from Fedora with Xfce immediately 😉