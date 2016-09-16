---
id: 55
title: 'Solving: Kernel panic &#8211; not syncing: VFS: Unable to mount root fs on unknown-block(x,y)'
date: 2013-02-03T22:22:27+00:00
author: Björn Bilger
layout: post
guid: http://yabapal.wordpress.com/?p=55
permalink: /2013/02/03/solving-kernel-panic-not-syncing-vfs-unable-to-mount-root-fs-on-unknown-blockxy/
categories:
  - Linux
  - Raspberry Pi
tags:
  - boot
  - file system
  - kernel panic
  - Linux
  - Raspberry Pi
---
Chances are high that pulling the plug off your Raspberry Pi (RPi) without shutting it down, will corrupt your file system. This post is about how to fix the problem, without re-installing your entire system.<!--more-->

Yesterday I changed the cabling of my media center and forgot to shut down my RPi properly via ssh. And this corrupted my file system. After trying to boot my RPi again, I saw this nice error message:

> Kernel panic &#8211; not syncing: VFS: Unable to mount root fs on unknown-block(x,y)

I tried to get some information about the error in the context of RPi. Most people suggested to re-install the entire system. Even though it doesn&#8217;t take too much time to re-install the system, re-configuring it does. Since re-installing the system wasn&#8217;t an option for me, I searched for other solutions and found the tool &#8220;fsck.ext4&#8221;.

I disconnected the USB stick from my RPi, connected it to my Linux-PC and ran the following command:

{% highlight bash linenos %}
{% raw %}
#replace sdd1 with your specific device
fsck.ext4 -y /dev/sdd1
{% endraw %}
{% endhighlight %}

This fixed the file system and I was able to boot my RPi with that USB stick, again.

If you installed your system to your RPi&#8217;s SD card, the solution should work, too, but requires a SD card reader, of course.

In order to avoid the problem, you should always shutdown your RPi via ssh:

``` bash
sudo shutdown -h now
```

If you can not connect via ssh, another solution might be to buy an USB stick with a built-in LED, indicating read/write access. If the LED is off, it should be possible to pull the plug relatively safely. This method, however, is not very reliable. You should use it only in case the first method is not possible. I only use it, in case the RPi&#8217;s ssh daemon died or is inaccessible for another reason.
