HTPC
====

Because cable TV is ridiculous.

There are many options available to setup a home entertainment system which plays media from a computer. This is how I've been doing it using open-source Linux-based technologies since about 2009. 

I use a [quiet PC](http://en.wikipedia.org/wiki/Quiet_PC) running Linux as an
[HTPC](http://en.wikipedia.org/wiki/Htpc) for my home entertainment system. The flow for new media starts with a torrent file downloaded from any computer. The torrent files are scp'd to a [seedbox](http://en.wikipedia.org/wiki/Seedbox) for quick downloading. A cron job running on the HTPC securely syncs the downloaded media. Once downloaded, DLNA is used to make the media available for consumption via the stereo or TV. This seems simple, although unfortunately the setup can be time consuming and a bit intimidating depending on your comfort level with Linux. Hopefully this repository will provide some assistance to those interested in a similar setup.

![htpc schema](htpc/raw/master/htpc.png "HTPC Schema")

# HTPC Linux Server #
This is the brain of the operation. I use an outdated MSI Wind PC running
Ubuntu Server with WD Caviar Green drives. More modern hardware options include the MSI Wind Box or Asus Eee Box. Just about anything that runs whisper quiet and sports a reasonably sized hard disk should do the trick. 

The following are the services running on the machine.

## DLNA ##
Explaining [DLNA](http://en.wikipedia.org/wiki/Digital_Living_Network_Alliance) is outside the scope of this document, but having a basic
understanding of the terms will be helpful. The [official site provides a good start](http://www.dlna.org/dlna-for-industry/digital-living/how-it-works/dlna-device-classes/digital-media-server). I find the definitions confusing, I'm not even positive if I'm labeling things properly below, so feel free to call me an idiot if you're a DLNA wizard.

## DLNA Digital Media Server (DMS) ##
A DLNA DMS makes the files on the HTPC server available to Digital Media
Player (DMP) and Digital Media Controller (DMC) devices. 

### Mediatomb ###
[MediaTomb](http://mediatomb.cc/) is my primary DMS for video. Any media rsync'd from the seedbox will be available immediately thanks to the wb_rsync script discussed below. In the examples below, MediaTomb would read from /localmedia/video

### MiniDLNA ###
[MiniDLNA](http://sourceforge.net/projects/minidlna/) is a bare-bones media
server which also works well. Sometimes it's nice to have two separate servers,
which poses no problems as long as they run on different ports.

### Squeezebox Server ###
My mp3 collection is streamed to the stereo with an outdated [Squeezebox Duet](http://www.logitech.com/en-us/support/speakers-audio/3817). The current product is the [Squeezebox Touch](http://www.logitech.com/en-us/speakers-audio/wireless-music-systems/squeezebox-touch). I won't go through the setup here, but the server runs the [Logitech Media Server](http://en.wikipedia.org/wiki/Logitech_Media_Server) software which indexes and makes your music available to the Squeezebox hardware, which connects to your receiver. Squeezebox Server is also a DLNA DMS, although I don't use it in this mode.

The squeezebox software points at my mp3 archive location and additionally the rsync'd download location (/localmedia/downloaded), so all mp3 downloads are available on my stereo shortly after they are rsync'd. 

## Continuous rsync script - wb_rsync ##
This script is run out of cron to continuously sync all media downloaded from the Seedbox. The script source is uploaded to this repo. The cron job below uses setlock from the daemontools package to prevent process stacking / overlap.

	*/14 * * * * /usr/bin/setlock -nX ~/var/run/wb_rysnc.lock ~/bin/wb_rsync 2>&1 >> /var/log/wb_rsync.log 

This script downloads content from the seedbox and places it in the
/localmedia/video directory, for access by MediaTomb.

## DYNDNS ##
Although unnecessary, I find it useful to have remote access to my HTPC server. I use http://zoneedit.com and the [zoneclient.py](http://zoneclient.sourceforge.net/) script to make sure my DNS record remains up to date if my ISP decides to change my assigned IP. Here is the crontab:

    */29 * * * * /usr/bin/python ~/opt/zoneclient/zoneclient.py --syslog -d ~/opt/zoneclient -r http://dynamic.zoneedit.com/checkip.html --acctfile ~/opt/zoneclient/acctinfo


# Seedbox #
While technically you could get away with running a torrent client on the HTPC
itself, for a multitude of reasons, using a [seedbox](http://en.wikipedia.org/wiki/Seedbox) is a good idea. 

There are a ton of Seedbox options. I have had great luck with [Whatbox](https://whatbox.ca/). They provide some nifty web interfaces including ruTorrent.

## rTorrent ##
[rTorrent](http://libtorrent.rakshasa.no/) is an ncurses based torrent client. When torrents are uploaded to the seedbox, they should go into a watched directory (~/torrents), so rTorrent will pick them up and begin downloading (to ~/seeding) automatically. I recommend [enabling encryption](https://wiki.archlinux.org/index.php/RTorrent#Additional_settings).

There are a couple of good rTorrent guides available, here is one: http://fsk141.com/rtorrent-the-complete-guide

## irssi ##

Some torrent trackers offer an IRC feed of their announce list. You can use [irssi](http://irssi.org/) to run a bot in these channels and auto-download things you are interested in.


# Downloading torrents using any computer #
With this HTPC setup, you can download torrents from any computer and 
easily shoot them out to your seedbox for downloading and auto-sync, then check
up on the progress on your HTPC. Here are a
couple of command aliases I've found helpful:

    alias scp_tor='scp ~/Downloads/*.torrent username@seedbox:~/torrents && rm ~/Downloads/*.torrent'
	alias twbs='ssh htpc tail -f /var/log/wb_rsync.log'

## Torrent Downloads ##
There are many torrent trackers available that offer torrents with free and legal media content. There are also others.. 

 * http://vodo.net
 * http://www.dimeadozen.org/
 * http://bt.etree.org
 * http://www.clearbits.net
 * http://linuxtracker.org
 * http://publicdomaintorrents.net

# DLNA Digital Media Player (DMP) #

A DMP finds and receives a DLNA stream from a digital media server (DMS) and converts it to standard audio or video signals which can be consumed by any TV or receiver. 

If you have a DLNA-enabled TV or A/V receiver, it may act as a DMP and a DMR, pulling content straight from a DMS and making a separate DMP unnecessary. I don't use one of these (for a couple different reasons). I assume that eventually high quality dual-use devices will become affordable enough to eliminate the need for a separate DMP, but this hasn't happened yet in my opinion.

I use the following DMPs:

## WD Live TV Plus ##
This [device](http://wdc.com/en/products/products.aspx?id=320) converts DLNA
streams, attached storage files, and other internet-based media to HDMI to be
consumed by your TV. 

## Squeezebox ##
See Squezebox Server above - this is the hardware component. The
[Logitech Squeezebox systems](http://www.logitech.com/en-us/speakers-audio/wireless-music-systems) are dedicated music streaming devices with accompanying software. While the software is not as slick as iTunes, it is open source software and runs on Linux.

