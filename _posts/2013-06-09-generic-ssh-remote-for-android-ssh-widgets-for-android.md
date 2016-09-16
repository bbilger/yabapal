---
id: 105
title: Generic ssh remote for Android / ssh widgets for Android
date: 2013-06-09T21:41:23+00:00
author: Björn Bilger
layout: post
guid: https://bbilger.com/yabapal/?p=105
permalink: /2013/06/09/generic-ssh-remote-for-android-ssh-widgets-for-android/
categories:
  - Android
  - Linux
  - Raspberry Pi
tags:
  - Android
  - automation
  - Linux
  - Raspberry Pi
  - remote
  - ssh
  - widgets
  - xbmc
---
# Intro

This guide will show you how to create a generic remote control for sending ssh commands via Android widgets. Generic means that you have full control over the layout, since you can order the widgets &#8211; as usual &#8211; in whichever position you like. There are a couple of apps in the Play Store that are able to send ssh commands, but I don't like them because of the design and the layout restrictions. So the main advantage of the appraoch I will show you is that you have full control over the layout. The disadvantage of the approach is that response time is not very good (2-5sec) since a new ssh connection will be created each time you execute a command &#8211; and destroyed when the command has been executed. If this is not an issue for you, and you like the idea, then this guide is for you.

This guide will use the apps  [Tasker](https://play.google.com/store/apps/details?id=net.dinglisch.android.taskerm) (2,99€) and [Tasker SSH Command Launcher](https://play.google.com/store/apps/details?id=com.aledthomas.taskersshcommand) (1,17€) to accomplish all that.

Leaving aside the general aspect of the guide, I will also provide information on how to connect  with ssh from Android using public/private key instead of using a password and information on how to execute (remote) commands that control XBMC. Regarding the last point, you then can create or rather design your own Android-based XBMC remote control.

This guide is based on Linux.

Like most of my posts, this is not a out-of-the-box solution and requires some knowledge regarding Linux, ssh, bash, etc., as well as some time.

<!--more-->

Just to give you an impression, here is a screenshot of my current layout, I control my media center with, when I don't expect immediate response.

[<img class="alignnone  wp-image-124" alt="24947" src="http://i2.wp.com/bbilger.com/yabapal/wp-content/uploads/2013/06/24947.png?resize=288%2C461&#038;ssl=1" data-recalc-dims="1" />](http://i2.wp.com/bbilger.com/yabapal/wp-content/uploads/2013/06/24947.png?ssl=1)

Note: I blurred the icons due to copyright. The icons show icons of radio streams I listen to and a click on the icon would start the  stream.

# History

A while ago I purchased my Rasperry Pi (RPi) and I am using it as my media center ([Raspbmc](http://www.raspbmc.com/)), together with my NAS. So I am watching movies and TV, listen to music and streams and so on. If my PC is not up and I can't send any commands via ssh, I always have to power on my TV. This is quite annoying. And which is most annoying is that I always execute some command from my Android device via ConnectBot, before I go to bed, in order start a timer to stop music playback. (I fall to sleep with music, best 😉 )

Since I hate doing the same over and over again (login via ConnectBot, type some command), I tried several ssh remote controls, but I did not like one of them. So I started to think that it would be nice to  do the same with widgets. Just before I started to develop this myself, I found [Tasker SSH Command Launcher](https://play.google.com/store/apps/details?id=com.aledthomas.taskersshcommand) which does exactly what I wanted. Even though the setup has some disadvantages, namely the delay, I can live with this. When I need better response times, I connect to my RPi via ssh directly or use Yatse.

This is just what I am using for. It is not restricted to RPi/XMBC. You can execute whatever commands via ssh, you like.

# Setup

## Apps

Install [Tasker](https://play.google.com/store/apps/details?id=net.dinglisch.android.taskerm) and [Tasker SSH Command Launcher](https://play.google.com/store/apps/details?id=com.aledthomas.taskersshcommand) on your android device.

Open [Tasker](https://play.google.com/store/apps/details?id=net.dinglisch.android.taskerm), switch to the &#8220;Tasks&#8221; tab, click on the plus sign and enter some name for your task.

[<img class="alignnone  wp-image-118" alt="24923" src="http://i2.wp.com/bbilger.com/yabapal/wp-content/uploads/2013/06/24923.png?resize=288%2C461&#038;ssl=1" data-recalc-dims="1" />](http://i2.wp.com/bbilger.com/yabapal/wp-content/uploads/2013/06/24923.png?ssl=1)

In the next screen, select Plugin and then SSH Command. Then you can set up the command by clicking on the Edit button. The screen shows one of my tasks and should be self explanatory. You need to provide a password only in case you stick to the User/Pass Auth Type. The section &#8220;Connection via public/private key&#8221; below provides the necessary steps to use a key file (Auth Type = Keyfile).

[<img class="alignnone  wp-image-120" alt="24948" src="http://i0.wp.com/bbilger.com/yabapal/wp-content/uploads/2013/06/24948.png?resize=288%2C461&#038;ssl=1" data-recalc-dims="1" />](http://i0.wp.com/bbilger.com/yabapal/wp-content/uploads/2013/06/24948.png?ssl=1)

Once you are done, click on &#8220;Done&#8221; in the upper right corner. In the next screen click on the &#8220;Action Edit&#8221; button in the upper left corner.

In the next screen, click on the grid in the lower right corner, in order to select an icon (you can use your own, built-in ones, application icons, etc.).

Then you need to click on the Task button in the upper left corner. Else [Tasker](https://play.google.com/store/apps/details?id=net.dinglisch.android.taskerm) might complain in the next step.

[<img class="alignnone  wp-image-121" alt="24930" src="http://i0.wp.com/bbilger.com/yabapal/wp-content/uploads/2013/06/24930.png?resize=288%2C461&#038;ssl=1" data-recalc-dims="1" />](http://i0.wp.com/bbilger.com/yabapal/wp-content/uploads/2013/06/24930.png?ssl=1)

Now you can go to the widget selection page and drag the Task widget on one of your home screens.

[<img class="alignnone  wp-image-122" alt="24928" src="http://i1.wp.com/bbilger.com/yabapal/wp-content/uploads/2013/06/24928.png?resize=288%2C461&#038;ssl=1" data-recalc-dims="1" />](http://i1.wp.com/bbilger.com/yabapal/wp-content/uploads/2013/06/24928.png?ssl=1)

This will open [Tasker](https://play.google.com/store/apps/details?id=net.dinglisch.android.taskerm) again. Select your task from the list (e.g. Stop 30). Make sure to click on &#8220;Task&#8221; in the upper right corner, again. If you click on Cancel, Back or Home, the widget won't get created.

If everything went well, you'll now have the task&#8217;s icon on one of your home screens and you should be able to execute your specified command on your remote machine. Don&#8217;t forget that it takes some seconds until the command gets send.

[<img class="alignnone  wp-image-123" alt="24931" src="http://i0.wp.com/bbilger.com/yabapal/wp-content/uploads/2013/06/24931.png?resize=288%2C461&#038;ssl=1" data-recalc-dims="1" />](http://i0.wp.com/bbilger.com/yabapal/wp-content/uploads/2013/06/24931.png?ssl=1)

## Central script directory (optional)

Even though you can provide a command directly in [Tasker SSH Command Launcher](https://play.google.com/store/apps/details?id=com.aledthomas.taskersshcommand), I like to have the scripts and so the commands at a central place. So I am also able to execute those scripts easily from my PC and don't need to provide them twice. Personally I place them under _~/scripts/remote_ but this is up to you, of course.

## Scripts for XBMC

If you want to use all this to execute commands for XMBC, then here are some infos:

First of all make sure to install _xbmc-send_ (on Debian-based distribution: _sudo apt-get install xbmc-send_).

With _xbmc-send_ you are able to execute any of the commands provided by XBMC. A list of available commands or rather built-in functions can be found here: [http://wiki.xbmc.org/index.php?title=List\_of\_built-in_functions](http://wiki.xbmc.org/index.php?title=List_of_built-in_functions)

The syntax for executing a command is:

<pre class="brush: bash; title: ; notranslate" title="">xbmc-send --action="&lt;some command&gt;"
</pre>

Here are two scripts I am currently using:

The first script stops playback after a given time:

<pre class="brush: bash; title: ; notranslate" title="">#!/bin/bash
stop_time="$1"
if [ "x$1" = "x" ]; then
         stop_time="0"
fi
echo 'xbmc-send --action="PlayerControl(Stop)"' | at now+"$stop_time"min
</pre>

So in case of the example given in this guide, I pass 30 as a parameter to the script and playback would stop after 30 minutes.

The second script is a generic script for playing any type of media.

<pre class="brush: bash; title: ; notranslate" title="">#!/bin/bash
cd $(dirname $0)
./stop 0
sleep 1
xbmc-send --action="PlayMedia($1)"
</pre>

I am using it for example to start a radio streams (.pls).

You might wonder about line 3 and 4 in the script. If I do not stop playback and don't wait for a second, then my RPi or rather [Raspbmc](http://www.raspbmc.com/) will crash. So this is a workaround to avoid a crash by delaying playback of the next resource. It might be that you don&#8217;t have those problems and maybe it&#8217;s specific to [Raspbmc](http://www.raspbmc.com/), only.

In case you want to execute some plugin functionality, remotely, then you should check ~/.xmbc/xbmc.log. This is especially usefuly, when you want to know which streams or resource is opened. Check for this line &#8220;COMXPlayer: Opening: &#8230;&#8221;.

Note: If you only want to use all this for controlling XBMC, then instead of using ssh, you could also use [XBMC's REST API](http://wiki.xbmc.org/index.php?title=JSON-RPC_API) and utilize the [Tasker](https://play.google.com/store/apps/details?id=net.dinglisch.android.taskerm) plugin [RESTask for Tasker](https://play.google.com/store/apps/details?id=com.freehaha.restask) to access it.

## Connection via public/private key (optional)

This is an optional step, because you can also provide username and password in [Tasker SSH Command Launcher](https://play.google.com/store/apps/details?id=com.aledthomas.taskersshcommand).

However, if you don't want to provide your password and/or connect to your ssh server quite often, I recommend this step.

### Create a public/private key-pair

Just execute the following command from a terminal, in order to create a public/private key-pair. You can do this on your local machine.

<pre class="brush: bash; title: ; notranslate" title="">ssh-keygen -t rsa
</pre>

Just follow the instructions&#8230;

  1. <span style="line-height: 13px;">Enter a file path for your private key (e.g. ~/.ssh/rpi)</span>
  2. If you like you can also enter a password, but this is not required.

This will generate two files: a private key wihout any extension (e.g. ~/.ssh/rpi) and a public key (e.g. ~/.ssh/rpi.pub).

### Using that key from ConnectBot

If you like to, you can use that key, in ConnectBot, too.

Copy your private key (e.g. rpi) to your Android device's &#8221; mounting root&#8221; folder (the folder you see, when mounting your device to your PC).

Start ConnectBot and import your private key via: Menu->&#8221;Manage Pubkeys&#8221;->Menu->&#8221;Import&#8221; and select your private key from the list (e.g. rpi).

### Install the public key on your remote server

In case, you don't have a ssh server installed, install for example openssh. On a Debian-based distribution: _sudo apt-get install openssh-server_

  1. <span style="line-height: 13px;"><span style="line-height: 13px;">If your ssh server is on a remote machine, connect to it (<em>ssh <usernam>@<ip></em>)</span></span>&nbsp;
  2. Check if the folder _~/.ssh and the file_ _~/.ssh/authorized_keys_ exist (not the case on RPi). If not: Create the folder and the file with proper permissions: _cd ~ && mkdir .ssh && chmod 700 .ssh && touch .ssh/authorized\_keys && chmod 600 .ssh/authorized\_keys_
  3. Append your previously created public key to the authorized_keys file: _echo &#8220;CONTENTS OF YOUR PUBLIC KEY FILE, entered via clipboard&#8221; >> ~/.ssh/authorized_keys _or copy your public key to the server and perform: _cat /path/to/your/public/key >>  ~/.ssh/authorized_keys_

Now you should be able to connect to your ssh server from ConnectBot (if you imported the key) and from [Tasker SSH Command Launcher](https://play.google.com/store/apps/details?id=com.aledthomas.taskersshcommand), without providing your password.