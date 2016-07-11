---
layout: post
title: V-USB to NRF24L01+ Adapter Part 2 - Assembly
bigimg: /img/vusb-nrf24-01.jpg
---

My PCBs arrived just twelve days after I ordered them and I'm sure you've all heard of OSHPark's spectacular service. All three boards are perfect in every way, even under a microscope I can't see any alignment issues or defects.  
What I failed to notice was that I have to swap MISO and MOSI when using the AtTiny's USI as an SPI master, but that's nothing a little bodge can't fix.  
![bodge](vusb-nrf24-bodge.jpg)  
Soldering was pretty straightforward and after the error was fixed, both the PCB and software worked right out of the box.  
Now what's left to do is to write the daemon for the PC and an alternative HID firmware for the dongle.
