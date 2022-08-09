---
layout: page 
---

> **Caution!**  
  This tutorial describes pocketsphinx [5 pre-alpha release](https://sourceforge.net/projects/cmusphinx/files/pocketsphinx/5prealpha/).
  It is not up-to-date with the current development version and will be updated soon.

Install is simple, you need to setup and properly configure alsa, then you can 
just build and run pocketsphinx

	
	sudo apt-get update
	sudo apt-get upgrade
	cat /proc/asound/cards


check your microphone is visible or not and if on which usb extension

     sudo nano /etc/modprobe.d/alsa-base.conf


Now change this

     # Keep snd-usb-audio from being loaded as first soundcard 
     options snd-usb-audio index=-2

To
     options snd-usb-audio index=0

if there is some other options `snd-usb-audio index=1`, comment it out

     sudo reboot 
     cat /proc/asound/cards 
     check your device is at 0
     sudo apt-get install bison
     sudo apt-get install libasound2-dev

download sphinxbase latest , extract
     
     ./configure --enable-fixed
     make
     sudo make install

download pocketsphinx, extract
     
     ./configure
     make
     sudo make install
     export LD_LIBRARY_PATH=/usr/local/lib 
     export PKG_CONFIG_PATH=/usr/local/lib/pkgconfig

Run it, should work

     pocketsphinx_continuous -inmic yes

