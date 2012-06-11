HTPC
====

Because cable TV is ridiculous.

There are a variety of options available to setup a home entertainment system which plays media from a computer. The following is a description of the setup I've been using since about 2009, which focuses on open-source Linux-based technologies.

Use a [quiet PC](http://en.wikipedia.org/wiki/Quiet_PC) running Linux as an [HTPC](http://en.wikipedia.org/wiki/Htpc). The flow for new media starts with a torrent file downloaded from any computer. The torrent files are then scp'd to a [seedbox](http://en.wikipedia.org/wiki/Seedbox) for quick downloading. A cron job running on the HTPC securely syncs the downloaded media. Once downloaded, DLNA is used to make the media available for consumption via a stereo or TV. 

The setup should be straight forward, although unfortunately it can be time consuming and a bit intimidating depending on your comfort level with Linux. Hopefully this repository will provide some assistance to those interested in a similar setup.

* [HTPC Linux Server](#htpc-linux-server)
    * [DLNA](#dlna)
    * [DLNA Digital Media Server (DMS)](#dlna-digital-media-server-dms)
        * [Mediatomb](#mediatomb)
        * [MiniDLNA](#minidlna)
        * [Squeezebox Server](#squeezebox-server)
    * [Continuous rsync script - wb_rsync](#continuous-rsync-script---wb_rsync)
    * [DYNDNS](#dyndns)

* [Seedbox](#seedbox)
    * [rTorrent](#rtorrent)
    * [irssi](#irssi)

* [Downloading torrents using any computer](#downloading-torrents-using-any-computer)
    * [Torrent Downloads](#torrent-downloads)

* [DLNA Digital Media Player (DMP)](#dlna-digital-media-player-dmp)
    * [WD Live TV Plus](#wd-live-tv-plus)
    * [Squeezebox](#squeezebox)

![htpc schema](htpc/raw/master/htpc.png "HTPC Schema")

# HTPC Linux Server #
This is the brain of the operation. I use an outdated MSI Wind PC running Ubuntu Server with WD Caviar Green drives. More modern hardware options include the MSI Wind Box or Asus Eee Box. Just about anything that runs whisper quiet and sports a reasonably sized hard disk should do the trick. 

The following services or similar alternatives should run on the machine.

### DLNA ###
Explaining [DLNA](http://en.wikipedia.org/wiki/Digital_Living_Network_Alliance) is outside the scope of this document, but having a basic
understanding of the terms will be helpful. The [official site provides a good start](http://www.dlna.org/dlna-for-industry/digital-living/how-it-works/dlna-device-classes/digital-media-server). The definitions can be confusing, and I'm not sure if things are labeled properly below, so feel free to call me an idiot if you're a DLNA wizard.

### DLNA Digital Media Server (DMS) ###
A DLNA DMS makes the files on the HTPC server available to Digital Media
Player (DMP) and Digital Media Controller (DMC) devices. 

##### Mediatomb #####
[MediaTomb](http://mediatomb.cc/) is a solid DMS. In the setup described here, it is used primarily for video. Any media rsync'd from the seedbox will be available immediately thanks to the wb_rsync script. In the examples below, MediaTomb is setup to scan /localmedia/video for new media.

##### MiniDLNA #####
[MiniDLNA](http://sourceforge.net/projects/minidlna/) is a bare-bones media
server which also works well. Sometimes it's nice to have two separate servers,
which poses no problem as long as they run on different ports.

##### Squeezebox Server #####
I stream my mp3 collection to the stereo with an outdated [Squeezebox Duet](http://www.logitech.com/en-us/support/speakers-audio/3817). The current product is the [Squeezebox Touch](http://www.logitech.com/en-us/speakers-audio/wireless-music-systems/squeezebox-touch). The HTCP server runs the [Logitech Media Server](http://en.wikipedia.org/wiki/Logitech_Media_Server) software which indexes and makes music files available through the Squeezebox hardware, which connects to your A/V receiver. Squeezebox Server is also a DLNA DMS.

The squeezebox software can additionally point to the rsync'd download location (/localmedia/downloaded), so all mp3 downloads can be available on the stereo shortly after they are rsync'd. 

### Continuous rsync script - wb_rsync ###
This script is run out of cron to continuously sync all media downloaded from the seedbox. The script source is uploaded to this repo. The cron job below uses setlock from the daemontools package to prevent process stacking / overlap.

	*/14 * * * * /usr/bin/setlock -nX ~/var/run/wb_rysnc.lock ~/bin/wb_rsync 2>&1 >> /var/log/wb_rsync.log 

The example script downloads content from the seedbox to /localmedia/downloaded, and also unpacks and rar archives (common for video torrents) and places the video files in the /localmedia/video directory, for access by MediaTomb.

### DYNDNS ###
Although unnecessary, it can be useful to have remote access to the HTPC server. I use http://zoneedit.com and the [zoneclient.py](http://zoneclient.sourceforge.net/) script to make sure the DNS record remains up to date if your ISP decides to change your assigned IP. Here is the crontab entry:

    */29 * * * * /usr/bin/python ~/opt/zoneclient/zoneclient.py --syslog -d ~/opt/zoneclient -r http://dynamic.zoneedit.com/checkip.html --acctfile ~/opt/zoneclient/acctinfo

# Seedbox #
While technically you could get away with running a torrent client on the HTPC
itself, for a multitude of reasons, using a [seedbox](http://en.wikipedia.org/wiki/Seedbox) is a good idea. 

There are a ton of seedbox options. I have had great luck with [Whatbox](https://whatbox.ca/). They provide some nifty web interfaces including ruTorrent.

### rTorrent ###
[rTorrent](http://libtorrent.rakshasa.no/) is an ncurses based torrent client. When torrents are uploaded to the seedbox, they should go into a watched directory (~/torrents). This means rTorrent will detect new torrents and begin downloading (to ~/seeding) automatically. [Enabling encryption](https://wiki.archlinux.org/index.php/RTorrent#Additional_settings) is recommended.

There are a couple of good rTorrent guides available, here is one: http://fsk141.com/rtorrent-the-complete-guide

### irssi ###

Some torrent trackers offer an IRC feed of their announce list. You can use [irssi](http://irssi.org/) to run a bot in these channels and auto-download things you are interested in.


# Downloading torrents using any computer #
With this HTPC setup, you can download torrents from any computer and 
easily transfer them out to your seedbox for downloading and auto-sync. You can also check up on the progress on your HTPC. Here are a couple of helpful command aliases:

    alias scp_tor='scp ~/Downloads/*.torrent username@seedbox:~/torrents && rm ~/Downloads/*.torrent'
	alias twbs='ssh htpc tail -f /var/log/wb_rsync.log'

### Torrent Downloads ###
Many trackers offer [pirated content](http://theoatmeal.com/comics/game_of_thrones), and private ones usually do so without the ads. There are also a few torrent trackers which feature torrents to free and legal media content:
 * http://vodo.net
 * http://www.dimeadozen.org/
 * http://bt.etree.org
 * http://www.clearbits.net
 * http://linuxtracker.org
 * http://publicdomaintorrents.net

# DLNA Digital Media Player (DMP) #

A DMP finds and receives a DLNA stream from a digital media server (DMS) and converts it to standard audio or video signals which can be consumed by any TV or A/V receiver. 

If you have a DLNA-enabled TV or A/V receiver, it may act as a DMP and a DMR, pulling content straight from a DMS and making a separate DMP unnecessary. I don't use one of these (for a couple different reasons). I assume that eventually high quality dual-use devices will become affordable enough to eliminate the need for a separate DMP, but this hasn't happened yet in my opinion.

I use the following DMPs:

### WD Live TV Plus ###
This [device](http://wdc.com/en/products/products.aspx?id=320) converts DLNA
streams, attached storage files, and other internet-based media to HDMI to be
consumed by your TV. 

### Squeezebox ###
See Squezebox Server above - this is the hardware component. The
[Logitech Squeezebox systems](http://www.logitech.com/en-us/speakers-audio/wireless-music-systems) are dedicated music streaming devices with accompanying software. While the software is not as slick as iTunes, it is open source software and runs on Linux.

