---
id: 18
title: How to automatically push content to Android devices
date: 2013-01-06T21:08:29+00:00
author: Björn Bilger
layout: post
guid: http://yabapal.wordpress.com/?p=3
permalink: /2013/01/06/how-to-automatically-push-content-to-android-devices/
tagazine-media:
  - 'a:7:{s:7:"primary";s:0:"";s:6:"images";a:0:{}s:6:"videos";a:0:{}s:11:"image_count";i:0;s:6:"author";s:8:"41859952";s:7:"blog_id";s:8:"42429198";s:9:"mod_stamp";s:19:"2013-01-06 20:08:29";}'
categories:
  - Android
  - Linux
tags:
  - Android
  - automation
  - Linux
  - offline reading
  - Pocket Alternative
---
# **What**

In this post I will show you how to automatically push content (websites, videos, music or rather files in general) from a Linux machine (the approach should be adaptable to windows/mac, as well) to an Android device (tablet or smartphone).

The setup might take some time, but if you finish this guide, you can simply grab your Android device in the morning and have the latest content you want, without doing anything! So it&#8217;s worth the time to set it up, once. Nevertheless: Some basic knowledge about Bash and Perl is necessary.

<!--more-->

# **Why**

A few weeks ago I finally received my Nexus 7. Unfortunately it is, like many tablets, a WiFi-only device. So I cannot use it on my way to work, to read or watch the latest news. In order to overcome this limitation, two solutions exist:

  1. tethering my smartphone to the tablet
  2. using an app like [Pocket (Formerly Read It Later)](https://play.google.com/store/apps/details?id=com.ideashower.readitlater.pro&hl=en)

I don&#8217;t like the first approach because the mobile network availability on my way to work is not very reliable and sometimes very slow. Furthermore it&#8217;s simply too much work early in the morning&#8230;

The disadvantage of the second is that I don&#8217;t want to select the articles I want to read, but the complete news sites. Furthermore the app requires registration, syncs via cloud, etc.

Fortunately at least one other solution exists &#8211; my solution 🙂

# **Abstract solution**

My solution to this problem, is to use [ADB (Android Debug Bridge)](http://developer.android.com/tools/help/adb.html). This little tool comes with the [Android SDK](http://developer.android.com/sdk/index.html) and allows you to interact with your Android device via a remote shell, push/pull files to and from your Android device, etc.

ADB can either be used when you connect your device to your PC via USB, but also, and that is important here, via WiFi.

So we will use ADB to push files at a scheduled time (cron or at start-up) from our PC to our Android device via WiFi.

Since for security reasons it is a bad idea to leave ADB-over-WiFi enabled all the time on your device, we need another cron job on our device, that enables and disables ADB for us.

Last but not least, we need some scripts that generate the content for us. I will cover how to fetch websites and videos.

# **Prerequisites**

Some prerequisites, that are necessary in order to use the solution:

  * A PC running with Linux, and some basic knowledge about it.
  * Some basic knowledge about Perl and Bash.
  * A **rooted** Android device. Else enabling/disabling ADB-over-WiFi won&#8217;t work.
  * The [Android SDK](http://developer.android.com/sdk/index.html)
  * The app [Cron4Phone](https://play.google.com/store/apps/details?id=com.aes.cron4phonefree&hl=en), which is necessary in order to enable/disable ADB-over-WiFi.
  * A static IP for your Android device within your local network. Normally you can achieve this by mapping your device&#8217;s MAC address (e.g. Settings->About *->Status: Wi-Fi MAX address) to a local IP via your router&#8217;s web-interface.
  * Some time, because it&#8217;s a &#8220;nerdy&#8221; solution 😉

# **Solution**

Let&#8217;s start setting up the solution.

## Setup on Android

We begin with the Android device.

Install the app [Cron4Phone](https://play.google.com/store/apps/details?id=com.aes.cron4phonefree&hl=en) and open it. Go to the tab &#8220;Tasks&#8221; and add a task via the menu in that view.

Create a task to start ADB-over-WiFi:

  1. Enter a name e.g. &#8220;start adb wifi&#8221;
  2. Check &#8220;Active&#8221;
  3. Enter a cron expression in the second text field. For example `0 7 \* \* 1,2,3,4,5`, in order to start the task on weekdays at 7 o&#8217;clock. You can find more information about the cron syntax [here at Wikipedia](http://en.wikipedia.org/wiki/Cron#CRON_expression).
  4. Enter the script below into the third text field. **Attention:** Make sure to avoid typos! Android&#8217;s word completion is a bit annoying here.
  5. Save the task.

``` bash
svc wifi enable
input keyevent 26
sleep 10
input keyevent 26
setprop service.adb.tcp.port 5555
stop adbd
start adbd
```

Snippet explanation: In the first line we enable WiFi. The next fakes pushing the power button and thereby wakes the device which is necessary in order to really enable WiFi. Then we wait 10 seconds and press the power button again. The last three lines enable the ADB service on port 5555. You can, however, change it to another port, if you like to.

Create a task to stop ADB-over-WiFi:

  1. Enter a name e.g. &#8220;stop adb wifi&#8221;
  2. Check &#8220;Active&#8221;
  3. Enter a cron expression in the second text field. For example `10 7 \* \* 1,2,3,4,5`, in order to stop WiFi and the ADB service ten minutes later, again. You can find more information about the cron syntax [here at Wikipedia](http://en.wikipedia.org/wiki/Cron#CRON_expression).
  4. Enter the script below into the third text field. **Attention:** Make sure to avoid typos! Android&#8217;s word completion is a bit annoying here.
  5. Save the task.

``` bash
setprop service.adb.tcp.port -1
stop adbd
start adbd
svc wifi disable
```


Snippet explanation: The first three lines disable the ADB service and the fourth disables WiFi, again. The script is simply the counterpart of the first one.

Finally switch to the &#8220;Cron&#8221; tab, check &#8220;Auto Restart&#8221; (enable jobs on reboot) and press &#8220;Start&#8221;. Pressing &#8220;Start&#8221; is necessary each time you add/enable a new task &#8211; just in case you want to add more cron jobs.

That&#8217;s it on the device. Let&#8217;s go on with the PC side.

## Setup on PC

Download the [http://developer.android.com/sdk/index.html](Android SDK) and extract it somewhere on your hdd.

The first necessary script is the one below. It simply tries to connect to the Android device and pushes all files/folders from the passed directory to it. After each run the pushed files will be deleted and backed up into another folder (`/_adb/_push_backup`) in the same directory until the next execution.

``` perl
#!/usr/bin/perl
use strict;
use warnings;

my $BACK_UP_FOLDER = "_adb_push_backup";

my $androidDeviceIdPort;
my $localPath;
my $remotePath;
my $output;
my $adb;
my $backupPath;

$androidDeviceIdPort = $ARGV[0];
$localPath = $ARGV[1];
$remotePath = $ARGV[2];
$adb = $ARGV[3];

# explain usage
if (!(defined $androidDeviceIdPort && defined $localPath && defined $remotePath))
{
 print "Usage: adb_push.pl &lt;androidDeviceIdPort&gt; &lt;localPath&gt; &lt;remotePath&gt; &lt;adb path&gt;\n";
 print "Example: perl /home/user/scripts/adb_push.pl \"192.168.2.101:5555\" \"/home/user/android_push\" \"/mnt/sdcard/Download\" \"/home/user/android-sdks/platform-tools/adb\"";

exit(1);
}
# enclose paths
$backupPath = "\"".$localPath."/".$BACK_UP_FOLDER."\"";
$localPath = "\"".$localPath."\"";
$remotePath = "\"".$remotePath."\"";

print "Trying to connect...\n";

# wait until we are connected to the device

do {
 $output = `$adb connect $androidDeviceIdPort 2&gt;&1`;
 print $output;
 sleep(10);
}while($output =~ m/unable/);

print "Connection established\n";

# remove backup folder that might exist from a previous execution, because we don't want to push the backup folder
system("cd $localPath && rm -rf $BACK_UP_FOLDER");

print "Pushing files...\n";
print "$adb push $localPath $remotePath\n";
system("$adb push $localPath $remotePath");

system("$adb disconnect $androidDeviceIdPort");

# backup currently pushed files

# create backup folder
system("mkdir $localPath/$BACK_UP_FOLDER");
# move files and folders to the backup folder
system("find $localPath -maxdepth 1 -mindepth 1 -not -name $BACK_UP_FOLDER -print0 | xargs -0 mv -t $backupPath");
```

Save the script e.g. under `$HOME/scripts/adb_push.pl` and make the script executable.

You can start the script as follows:

``` bash
perl /home/user/scripts/adb_push.pl 192.168.2.101:5555 /home/user/android_push /mnt/sdcard/Download /home/user/android-sdks/platform-tools/adb
```

The parameters are defined as follows:

  1. <span style="line-height: 14px;">path to the script above</span>
  2. (static) IP of your Android device and the port of the ADB service (ip:port)
  3. location of the folder that contains the content, you want to push
  4. The path on your Android device were you want to push your files to. **Attention**: This depends on your Android device! On my phone the path starts with `/mnt/sdcard/...`, whereas on my tablet it starts with `/sdcard/...`. You can find the right for you by opening a remote shell on your Android device via ADB. Then navigate to the location you want to push the files to and use that path.
  5. path to the ADB executable

_Note: I recommend to use fully qualified path names to start the script, because, depending on your method to start the script, the $HOME environment variable might not be set. _

_Note: The script is in a loop until it can connect to the device. So if you don&#8217;t want to waste CPU cycles, you can add some lines of code to stop execution after a few retries._

### Testing

Now it is time to test our setup.

PC side:

  1. <span style="line-height: 14px;">place a file into the folder from where you want to push files from</span>
  2. start the script on your PC as described above

Android side:

  1. disable WiFi
  2. open Cron4Phone
  3. navigate to the `Tasks` tab
  4. long press on the task that starts ADB-over-WiFi (e.g. &#8220;start adb over wifi&#8221;)
  5. confirm execution
  6. grant super user rights (always)
  7. wait **Attention:** there seems to be some bug in the app, because it needs longer than 10 seconds for the screen to come back and Android says that the app does not respond. Just ignore this and press &#8220;OK&#8221; or &#8220;Wait&#8221;. The problem only exists when executing it manually, but not when the app triggers execution.

After a few seconds the perl script should push the file and stop execution then. Verify that on your Android device. If you have any trouble here, then just leave a comment.

### Scheduling

Now that we made sure that the setup works, we need to execute the script automatically. There are two possibilities:

  1. <span style="line-height: 14px;">set up a cron job that executes the script at a scheduled time</span>
  2. execute the script on startup

The first one is the only possibility, if your PC is running all the time. The second scenario can either be realized by the window managers capability to execute scripts on startup or via init-scripts. Using your search engine of choice will help you to set this up 😉

_Note: If you execute at startup, be aware of the fact that you can boot your PC even when it is shut down. Further information about this can be found in the [MythTV Wiki](http://www.mythtv.org/wiki/ACPI_Wakeup#Using_.2Fsys.2Fclass.2Frtc.2Frtc0.2Fwakealarm)._

## Content Generation

Now that all should be set up, let&#8217;s take the final step: creating content! This next section will and only can give you a basic idea of how to fetch content.

### Websites

We are going to start with downloading entire websites (a.k.a. &#8220;crawling&#8221;) &#8211; at least to a certain depth. The software I am using for that purpose is &#8220;httrack&#8221;. The software is extremely powerful, and yet very tricky to use. Check the manpage or your search engine of choice for further information.

Since the parameters to crawl a page are very dependent on the page, I can only give you a few hints:

  1. <span style="line-height: 14px;">make sure to set a mobile user agent (&#8220;-F <mobile user agent>&#8221;)</span>
  2. if a dedicated mobile site is available, make sure to use it: `http://m. * `
  3. Play around with the depth to crawl. Don&#8217;t set the depth too high. This will result in a long crawling/pushing procedure and will also result in a high load for the site owner. &#8220;&#8211;depth 2&#8221; works quite well on most mobile sites, but probably you won&#8217;t be able to crawl/read articles that spread over several pages.
  4. &#8220;&#8211;depth 3&#8221; on the other hand can result in long crawling times. You can try to reduce the load by excluding links with certain patterns: &#8220;-\*blog\* -\*ticker\*&#8221;. This list, can be quite long, so you have to decide for yourself, whether it&#8217;s worth the effort to read multi-page articles.
  5. Search the internet or the httrack forum. If you are lucky you will find the perfect parameters to crawl your page.

Currently I am using the following script for crawling. I am publishing it here, in the hope you can use it as template for your solution. You simply have to adapt the URLs, the depths and the additional options. Finding the right values for the two last-mentioned parameters might take some time. So try&error, as always 😉 You can, however, call httrack directly.

``` bash
#!/bin/bash

OUTPUT="/home/user/android_push/"
# make sure to use a mobile agent, in order to get the mobile site
USER_AGENT="'Mozilla/5.0 (Linux; U; Android 0.5; en-us) AppleWebKit/522+ (KHTML, like Gecko) Safari/419.3'"
DEFAULT_DEPTH="2"
BACKUP_DIR="_adb_push_backup"

# use a unique key (["..."]) for each page you want to crawl

# value = page's url
declare -A SITE_URL_HASH=( ["site1"]="http://m.site1.com/"
 ["site2"]="http://m.site2.com"
 ["site3"]="http://www.site3.com" )

# define a different crawling depth for a page than the default one, which is set to 2
declare -A SITE_DEPTH_HASH=(["site1"]="3"
 ["site2"]="4" )

# define additional options for httrack
declare -A SITE_OPTIONS_HASH=( ["site3"]="-*ticker* -*blog*" )

cd "$OUTPUT"

# wait for network to be available
while true; do
 ping -c 1 google.com
 if [[ $? == 0 ]];
 then
 echo "network available."
 break;
 else
 echo "network is not available, waiting..."
 sleep 5
 fi
done

cd "$BACKUP_DIR"

# remove from backup folder
for dir in "${!SITE_URL_HASH[@]}"; do
 rm -rf "$dir"
done

cd "$OUTPUT"

# backup previously crawled websites and their links
for dir in "${!SITE_URL_HASH[@]}"; do
 mv "$dir" "$BACKUP_DIR"
done

# crawl websites
for dir in "${!SITE_URL_HASH[@]}"; do

site="${SITE_URL_HASH["$dir"]}"
 depth="${SITE_DEPTH_HASH["$dir"]}"
 options="${SITE_OPTIONS_HASH["$dir"]}"
 if [ -z ${depth} ]
 then
 depth="$DEFAULT_DEPTH"
 fi
 if [ -z ${options} ]
 then
 options=""
 fi
 httrack "$site" -O "$OUTPUT"/"$dir" -F "$USER_AGENT" --depth "$depth" $options & # "&" =&gt; child process =&gt; parallel dl
done

wait # wait until all crawlers have finished
```

In order to crawl the pages automatically, you can create a cron job for your crawler script or glue it together with the adb push script.

#### Additional Apps

In order to use the websites on your Android device you&#8217;ll need two additional Apps:

  1. A file browser in order to navigate to the files.
  2. A browser. Yes, you need a browser, because you can&#8217;t open HTML files from your device&#8217;s storage with the default browser. Most alternative browsers in contrast, allow you to open local HTML files. In addition you can bookmark those pages.

As an alternative you can navigate to the index.html file with &#8220;file://&#8230;&#8221; or &#8220;file:///&#8230;&#8221;, depends&#8230; Or you can use an app like [OpenHtml](https://play.google.com/store/apps/details?id=com.liolick.android.openhtml&hl=e).

### Videos / Livestreams

Downloading a video is very often as simple as downloading a resource on the internet and can be done with the tool [wget](http://en.wikipedia.org/wiki/Wget).

What I was interested in, however, was to record live streams. Since most streams come to you wrapped in flash, it&#8217;s sometimes a bit difficult to get the stream&#8217;s URL. Therefore you can either analyse the network traffic, or ask your search engine of choice. Then you can use a tool like [rtmpdump](http://en.wikipedia.org/wiki/Rtmpdump) or [mplayer](http://www.mplayerhq.hu/DOCS/HTML/en/streaming.html) to dump the stream. The problem is that the process is rather difficult and the dumps normally must be transcoded, because those dumps won&#8217;t play properly on mobile devices &#8211; at least on mine.

So it is easier and more efficient to use [VLC](http://en.wikipedia.org/wiki/VLC_media_player) in conjunction with the tool [FreetuxTV](ttp://code.google.com/p/freetuxtv/). VLC will be used to dump the stream and FreetuxTV is helpful to find stream URLs (also available here: [http://database.freetuxtv.net](http://database.freetuxtv.net)) and to find the options for VLC. The advantage of VLC is that it can dump much more streaming protocols.

Start FreetuxTV via the commandline and select the transcoding option of your choice on the &#8220;Recordings&#8221; tab in the preferences. Now start to record the stream you like to dump. You can see the stream&#8217;s URL and the VLC options in the command line, now.

Eample output:

> [LibVLC-Gtk] INFO : Playing <your stream>

> [LibVLC-Gtk] INFO : Using vlc options [:sout=#transcode{vcodec=theo,vb=800,scale=1,acodec=vorb,ab=128,channels=2}:duplicate{dst=std{access=file,mux=ogg,dst=&#8221;/home/user/android_push/<stream name><date>.ogg&#8221;},dst=display}]

We can now use this as a template to record a stream using vlc:

``` bash
cvlc -vvv &lt;your stream&gt; --sout '#transcode{vcodec=theo,vb=800,scale=1,acodec=vorb,ab=128,channels=2}:duplicate{dst=std{access=file,mux=ogg,dst="/home/user/android_push/&lt;name&gt;.ogg"}}'
```

Make sure to remove `#sout`,  `,dst=display` , `[` and `]` from the options and enclose the options for the vlc call with single quotes.

Update: Don&#8217;t use the stream&#8217;s URL from the command line, but from the stream&#8217;s properties. The reason is that asx-&#8220;streams&#8221; redirect you to another stream. The stream you were redirected to will be shown on the shell, but you cannot use it, since it contains some authentication information and will reject future connection attempts.

What&#8217;s missing, is the ability to stop the stream dumping after some time. A simple bash script `runfor` that can do this for us:

``` bash
#!/bin/bash

TIME=$1
shift # remove 1st param from args

"$@" & # start program
PID=$!
sleep $TIME # wait
kill $PID # kill the started program / process
wait
```

So the VLC call changes to this:

``` bash
runfor 10m cvlc -vvv &lt;your stream&gt; --sout '#transcode{vcodec=theo,vb=800,scale=1,acodec=vorb,ab=128,channels=2}:duplicate{dst=std{access=file,mux=ogg,dst="/home/user/android_push/&lt;name&gt;.ogg"}}'
```

Change the 1st parameter (10m) to whatever time, you like to dump the stream for. Finally you can add that call into your crontab.

# Final words

That&#8217;s it! Well, the article became a little bit longer than it was intended to, but I hope some of you will follow it anyway.

Furthermore I hope that the solution works for you and saves your time and money (you don&#8217;t need mobile internet) .

Feel free to leave a comment, if you like the solution, have problems with it, have some hints for improvement or want to share your scripts for content generation or pushing.

# Side Note

I used this setup on my desktop PC, whereas my PC booted automatically in the morning (check [MythTV Wiki](http://www.mythtv.org/wiki/ACPI_Wakeup#Using_.2Fsys.2Fclass.2Frtc.2Frtc0.2Fwakealarm) for information about how to boot automatically).

Since I purchased a Raspberry Pi, the scripts are running on that machine right now. The Android SDK doesn&#8217;t run on its ARM processor, but some guys at the xda-developers forum managed to compile ADB on the ARM processor: <http://forum.xda-developers.com/showthread.php?t=1924492> The binary is also linked in that thread: <http://forum.xda-developers.com/attachment.php?attachmentid=1392336&d=1349930509>