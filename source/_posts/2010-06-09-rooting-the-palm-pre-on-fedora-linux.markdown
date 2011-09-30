---
layout: post
title: "Rooting the Palm Pre on Fedora Linux"
date: 2010-06-09 19:47
comments: true
categories: [linux,fedora] 
---	
</p><p>So a good friend just gave me his old Palm Pre… for those of you who don’t know, the Pre runs WebOS, a linux backed system which is impressively intuitive and fun to use, though the hardware is a mite underpowered (don’t worry, we’ll fix that here…) <img class="wp-smiley" alt=":)" src="http://web.archive.org/web/20100619042256im_/http://www.badgerbait.net/wp-includes/images/smilies/icon_smile.gif"> </p>
<!--more-->
<p>Some very good people at <a href="http://web.archive.org/web/20100619042256/http://www.webos-internals.org/wiki/Main_Page">WebOS Internals</a> have released some great software for cracking open the full potential of the Palm Pre, and were kind enough to produce it in a jar file, so it runs on Linux, as well as Mac and Windows. Now the process may go a bit easier for other systems, but it took a few tries to figure it out on my Fedora PC, so I figured I’d jot down a quick set of instructions for others who may want to get full control of their Palm Pre without resorting to using “that OS”. Note that while this is written specifically for Fedora, it should apply to any Linux Distro.</p>
<p><em>Without further ado, BadgerBait Productions, LLC, Esquire Jr, Inc presents:</em></p>
<h2><span style="text-decoration: underline;"><strong>Rooting the Palm Pre on Fedora</strong></span></h2>
<p>﻿1) <strong>Get Novacom Driver</strong></p>
<p><strong>Download Pkg File</strong> <a href="http://web.archive.org/web/20100619042256/http://cdn.downloads.palm.com/sdkdownloads/1.4.1.427/sdkBinaries/palm-novacom_1.0.55_amd64.deb">64 bit</a> or <a href="http://web.archive.org/web/20100619042256/http://cdn.downloads.palm.com/sdkdownloads/1.4.1.427/sdkBinaries/palm-novacom_1.0.55_i386.deb">32 bit</a> (source: http://developer.palm.com/index.php?option=com_sdkdownload&amp;view=home )</p>
<p><strong>Extract the archive</strong> (source:http://www.webos-internals.org/wiki/MojoSDK_on_Fedora_11 ):</p>
<p>$ ar xv palm-novacom_1.0.46_i386.deb<br>
 $ tar -zxvf data.tar.gz<br>
 # mv opt/Palm /opt/.<br>
 # mv usr/local/bin/* /usr/local/bin/.<br>
 # mv usr/share/doc/palm-novacom /usr/share/doc/.</p>
<p>if 64 bit: <br>
 yum install libusb.i686</p>
<p>$ tar -zxvf control.tar.gz<br>
 # ./postinst</p>
<p><strong>Clean Up Package Files</strong></p>
<p>$ rm -fr debian-binary data.tar.gz control.tar.gz opt/ usr/ control md5sums postinst prerm</p>
<p>2) <strong>WebOS Quick Install</strong> (source: http://forums.precentral.net/canuck-software/228310-webos-quick-install-v3-14-a.html)</p>
<p>Download newest version of <a href="http://web.archive.org/web/20100619042256/http://filebin.ca/jpnk/WebOSQuickInstall.jar">WOQI</a> , personally i save it in ~/.palm/ to keep it out of the way.</p>
<p>3) <strong>WebOS Doctor</strong></p>
<p><span style="color: #ff0000;">**WARNING** Do NOT run WebOS Doctor directly, as it WILL WIPE YOUR DEVICE </span></p>
<p>This is where things started to go all wahoony shaped when I was following normal install instructions, so rather than letting WebOS Quick Install download the <a href="http://web.archive.org/web/20100619042256/http://www.webos-internals.org/wiki/Webos_Doctor_Versions">WebOS Doctor</a>, we’ll do it manually. Just follow the link and download the appropriate version for your device.<br>
 Download appropriate version into same folder as WOSQI, then go ahead and launch the WebOS Quick Instal from CLI as follows:</p>
<p>~/.palm $ java -jar WebOSQuickInstall.jar</p>
<p>The first time it launches, it’ll notice that you’ve got the WebOS Doctor, then become very contented and take a nap.</p>
<p>4) <strong>Developer Mode</strong></p>
<p>In order to gain appropriate access to the device for adding HomeBrew Apps and Patches, you need to unlock Developer Mode for the phone. Just ‘cuz they’re that damn cool, the Pre folks unlock Developer Mode with the <a href="http://web.archive.org/web/20100619042256/http://en.wikipedia.org/wiki/Konami_Code">Konami Code</a>. so… just start typing: <em>upupdowndownleftrightleftrightbastart</em></p>
<p>Turn it on, and reboot your phone.</p>
<p>After reboot, connect the phone to your pc via usb, choose just charge on the phone.</p>
<p>Run same command in terminal as above to launch WOSQI, this time it should open.</p>
<p>Check Tools -&gt; Device Manager -&gt; make sure your device is listed. If your device is not, look under File -&gt; Options, and make sure USB Device is selected.</p>
<p>Congrats! You just rooted your Palm Pre on Fedora Linux!</p>
<p>5) Getting Started</p>
<p>http://www.precentral.net/how-add-homebrew-apps-patches-and-themes</p>
<p><br class="spacer_"></p>
<p>I personally like MojoTracker the best, paired w/ the UberKernel and Govnah to get the speed fixed. Note that I found a HUGE difference in overheating by OC’ing the phone to 800MHz, but had no heat issues and indistinguishable performance using the 720MHz OC Kernel.</p>
<p><span style="color: #ff0000;">Please note that OverClocking should be done with caution and only by those who know what they’re doing. For Palm’s official stance on overclocking, see <a href="http://web.archive.org/web/20100619042256/http://developer.palm.com/blog/2010/03/a-statement-on-the-overclocking-patches/">http://developer.palm.com/blog/2010/03/a-statement-on-the-overclocking-patches/</a></span></p>
