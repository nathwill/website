---
layout: post
title: "Ubuntu 10.04: Lucid Print and File Server for Home Network"
date: 2010-03-14 19:53
comments: true
categories: [linux,ubuntu]
---
<p>Having officially moved to a new laptop, and having given away the desktop, i’ve been a little short on projects around here lately. I looked around at some media server options, thought about doing a media server build, but can’t see spending the money on it, particularly when there’s nothing to connect the HTPC to the televisions without setting up some thin clients,&nbsp; of which I have all of none.</p>
<!--more-->
<p>I DID have my old HP ze4400 laying around collecting dust and cat-fur though. Now I know that Lucid is <em>technically</em> still <strong>beta</strong>, but screw it, this notebook’s been BSOD’d and reformatted more times than I can remember, and my fingers are itchin’.</p>
<p><span style="text-decoration: underline;"><strong>The Goal: A <em>Cheap</em> Print and File Server Configured for a Home Network</strong></span></p>
<p>For a long time now I’ve wanted to set up some kind of Network Attached Storage (NAS) device, and I’ve looked at some pretty <a target="_blank" href="http://web.archive.org/web/20101023204022/http://shopping.yahoo.com/search?p=nas" title="Network Attached Storage | Yahoo! Shopping">neat options</a>, but couldn’t bring myself to spend the money on it when I’ve got so many spare parts laying around the house waiting to be brought back from exile. <span id="more-419"></span>I’ve also wanted to set up a print server, because when you’re mainly working on a laptop, and you’re not the home-office type, you’re probably hanging out with the laptop wherever’s comfortable, which is definitely not where the printer is. With the rapidly approaching release of the next Long Term Support (LTS) Release of <a target="_blank" href="http://web.archive.org/web/20101023204022/http://www.ubuntu.com/products/whatisubuntu/serveredition" title="Ubuntu Server Edition | Canonical">Ubuntu Server Edition</a>, now seemed like a good time to take a glimpse at the release-candidate. <img class="wp-smiley" alt=":)" src="http://web.archive.org/web/20101023204022im_/http://www.badgerbait.net/wp-includes/images/smilies/icon_smile.gif"> </p>
<p><span style="text-decoration: underline;"><strong>Install the Server</strong></span></p>
<p>The server installation process is incredibly simple, completing in about 20 minutes on my old hardware. I chose defaults for the network configuration (DHCP), then selected the software packages for print and file-server, and SSH for remote administration. Ubuntu uses <a target="_blank" href="http://web.archive.org/web/20101023204022/http://samba.org/" title="Samba">Samba</a> and <a target="_blank" href="http://web.archive.org/web/20101023204022/http://cups.org/" title="CUPS">CUPS</a> to these ends. Having prior experience with these, that’ll work just fine. Once installed, the system boots fine and after a little scouting with Nmap to get the DHCP-assigned IP of the server, I was able to sign in from my day-to-day notebook and start customization.</p>
<p><span style="text-decoration: underline;"><strong>Configure Networking (Static IP) and Firewall</strong></span></p>
<p>First thing to do in order for the server to be easily locatable on the home network, is to configure the network connection for a static IP. The two critical files involved are <strong>/etc/network/interfaces</strong> and <strong>/etc/resolv.conf</strong>. Before making any changes, as usual, be sure to make backup copies of the oriignal configuration files.</p>
<p><code> user@host:~$ sudo cp /etc/resolv.conf /etc/resolv.conf.original</code><code><br>
 user@host:~$ sudo cp /etc/network/interfaces /etc/network/interfaces.original </code></p>
<p>For resolv.conf, the auto-generated one will work fine, or if you’re a fan of openness, you may want to use OpenDNS servers. You should customize for your own circumstances, but here’s an example resolve.conf:</p>
<p><code>nameserver 208.67.222.222</code><br>
 <code>nameserver 208.67.220.220</code><br>
 <code>domain hsd1.or.comcast.net.</code><br>
 <code>search hsd1.or.comcast.net.</code></p>
<p>Next is the interfaces config, for this one we’ll need to know some details about the network environment, specifically the IP you want to set your server on, the netmask and the gateway (router) ip. On Linux, you can get this from a computer with an internet connection by entering ‘ifconfig’ at the command line, or ipconfig on Windows. Example /etc/network/interfaces file below:</p>
<p><code># The loopback network interface</code><br>
 <code>auto lo</code><br>
 <code>iface lo inet loopback</code></p>
<p><code># The primary network interface</code><br>
 <code>auto eth0</code><br>
 <code>iface eth0 inet static</code><br>
 <code> address 192.168.1.105</code><br>
 <code> netmask 255.255.255.0</code><br>
 <code> gateway 192.168.1.1</code></p>
<p>Once you’ve set up the network files, make sure to restart networking; ‘sudo /etc/init.d/networking restart’.</p>
<p>To configure the firewall, we want to first think of the services we need to have network access to, and for whom those services are intended. For my setup, the services in action are SSH, Samba Server (Files), and CUPS server (Print). To set up these services, I chose to use the default firewall utility in Ubuntu, ufw (uncomplicated firewall). Since this is for a home network, I’ll be restricting access to clients inside the LAN. SSH is run on port 22 by default, so we’ll punch a hole there and in port 631 for CUPS. For Samba, ufw has a built in configuration file for Samba that it can use to determine what ports Samba needs access to. Here’s how I set up the firewall.</p>
<p><code>user@host:~$ sudo ufw allow proto tcp from 192.168.1.0/24 to any port 22</code> <code><br>
 user@host:~$ sudo ufw allow proto tcp from 192.168.1.0/24 to any port 631</code> <code><br>
 user@host:~$ sudo ufw allow from 192.168.1.0/24 to any app Samba</code> <code><br>
 user@host:~$ sudo ufw enable &amp;&amp; sudo ufw default deny</code> <code><br>
 user@host:~$ :~$ sudo ufw status<br>
 Status: active<br>
 </code></p>
<p><code>To &nbsp;&nbsp; &nbsp;&nbsp;                        Action&nbsp;&nbsp;      From<br>
 </code> <code>--&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;                         ------</code><code> ----</code><br>
 <code>Samba&nbsp;&nbsp;&nbsp;                      ALLOW&nbsp;&nbsp;&nbsp;       192.168.1.0/24</code> <br>
 <code>22/tcp&nbsp;&nbsp;                     ALLOW&nbsp;&nbsp;&nbsp;       192.168.1.0/24</code> <br>
 <code>631/tcp&nbsp;                    ALLOW&nbsp;&nbsp;&nbsp;       192.168.1.0/24</code></p>
<p>Alright, so that’s network and firewall. Now let’s look at the configuration of Samba and CUPS.<br>
 <span style="text-decoration: underline;"><strong> </strong></span></p>
<p><span style="text-decoration: underline;"><strong>Samba Configuration</strong></span></p>
<p>For Samba, the relevant configuration file is /etc/samba/smb.conf. While the default configuration file is fairly big and contains a lot of extra configuration options, I chose to copy smb.conf to smb.conf.original, and set up a new smb.conf file from scratch, as I have pretty simple needs; i only need to share files and disk space on the local network.</p>
<p>Since I’m only allowing access to internal clients, I am choosing not to require user-names and passwords, and to allow guest access so that I don’t need to add any extra users.</p>
<p><code> [global]<br>
 workgroup = VALHALLA <br>
 netbios name = YGGDRASIL</code></p>
<p>#optional print sharing through Samba <br>
 #[printers] <br>
 #browsable = yes <br>
 #guest ok = yes</p>
<p>[landrive] <br>
 path = /home/nathan/share <br>
 guest ok = yes <br>
 comment = Home Network Storage <br>
 available = yes <br>
 public = yes <br>
 writable = yes <br>
 browsable = yes <br>
 create mask = 0666</p>
<p>After setting up smb.conf, be sure to restart Samba; ‘sudo /etc/init.d/smbd restart’. And of course you’d want to customize path for your own share. At this point you can test from any other computer on the network and you should be able to read and write files to the shared content. Also be sure to customize the create mask to your desired circumstances. I chose to give everyone read/write permissions, but 0644 might be a better idea depending on your circumstances. I recommend NOT making scripts executable by default, though I hope you can trust the people in your home…</p>
<p><span style="text-decoration: underline;"><strong>CUPS Configuration</strong></span></p>
<p>CUPS stands for the Common Unix Printing System, and is one area where Linux really stands out over Windows. Providing top-notch driver support and a slick and intuitive web-based administration, CUPS is really a joy to work with.</p>
<p><code> ServerAdmin nathan@localhost <br>
 # Allow remote access <br>
 Port 631 <br>
 Listen /var/run/cups/cups.sock <br>
 DefaultEncryption Never    <br>
 # Allow shared printing and remote administration... </code></p>
<p>Order allow,deny   <br>
 <code>Allow from </code><code>@LOCAL</code></p>
<p># Allow remote administration…   <br>
 Order allow,deny   <br>
 <code>Allow from </code><code>@LOCAL<br>
 </code><code><br>
 # Allow remote access to the configuration files... </code></p>
<p>Order allow,deny   <br>
 Allow from <code>@LOCAL</code></p>
<p><code><br>
 # Share local printers on the local network. <br>
 Browsing On <br>
 BrowseOrder allow,deny <br>
 BrowseAddress @LOCAL</code><code><br>
 BrowseAllow from @LOCAL<br>
 DefaultAuthType Basic </code></p>
<p>This configuration file will give share, browsing and administrative priviliges to local network clients. Remember to restart CUPS; ‘sudo /etc/init.d/cups restart’. Now that you’ve configured cups to allow remote administration, you can add the printer through the web-based interface, by visiting the web-administration console. The address will be similar to: http://ip.of.your.server:631/. For more information on CUPS, visit their website at http://www.cups.org/.</p>
<p><span style="text-decoration: underline;"><strong>Expanding Storage Capacity with USB-HDD</strong></span></p>
<p>Great, so now the server set-up is done, tested and confirmed file-sharing access and remote printing. spiff! But… this crappy old laptop only has a 30GB harddrive, whereas both of the new ones have 250GB harddrives, so unless we improve the storage somehow, this file-server’s not going to be worth very much. Fortunately I also managed to dig out my 1TB drive and external enclosure. While this isn’t the slickest solution, it’s cheap, and readily available, so wtf, I’m goin’ for it.</p>
<p>First the bad news. If your external storage device isn’t already formatted, do it from a Windows machine or from command line, because both GParted and Palimpsest lay it out as GPT, which for some god-forsaken reason isn’t supported without a kernel change in Ubuntu Server. If you haven’t formatted a harddrive from the command line before, there’s TONS of great tutorials online for basic linux sys-admin tasks, and a good <a target="_blank" href="http://web.archive.org/web/20101023204022/http://www.ehow.com/how_1000631_hard-drive-linux.html" title="How to Format a Hard Drive in Linux | ehow.com">one for this particular topic on ehow.com</a>. Don’t worry about Windows compatibility either, since we’re going through a Samba share, Windows won’t have to support it.</p>
<p>Now to the nitty gritty!</p>
<p>After hooking up the hard drive, one of the first things I noticed was that Ubuntu Server Edition doesn’t automatically mount hotplugged drives. Makes sense for servers I suppose, the less they do automatically, the fewer surprises one may encounter. So… mkdir a folder inside your already configured Samba share (or if you want somewhere new), then set-up the entry for the drive in /etc/fstab. Since the drive IS hotpluggable, I recommend setting it up to identify the drive by UUID. To get the UUID, you can find the drive by ‘ls /dev/sd*’, then run ‘tune2fs /dev/sdx’ where x is the USB-HDD drive. once you have the UUID, you can set up an entry for the drive in /etc/fstab. In my case I choose to mount the drive inside the existing share for simplicity’s sake. My fstab looks like:</p>
<p><code>UUID=63f5c4e1-3077-481c-bfb2-32</code><code>1a7</code><code>87fdc6f       /home/nathan/share/backup       ext4    defaults        0       0</code></p>
<p>To have the server automount the drive when detected, you can install usbmount ‘sudo apt-get install usbmount’. Now it’s set up to automount your extended storage capacity to a predetermined location every time it detects the drive.</p>
<p>An issue is raised in using an external HDD for storage capacity though, and the issue is that external harddrives really aren’t meant to be left on all the time. There’s a really great script available over at <a target="_blank" href="http://web.archive.org/web/20101023204022/http://blog.shadypixel.com/safely-removing-external-drives-in-linux/" title="Safely Removing External Drive | ShadyPixel.com">shadypixel</a> that will shut the hard drive entirely down, save it to the server and make it executable. To have it run every time the server powers off, just move the script to /etc/init.d/ and link to it from /etc/rc0.d/.</p>
