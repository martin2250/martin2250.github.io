---
layout: post
title: V-USB to NRF24L01+ Adapter Part 4 - HID
bigimg: /img/volumeknob.jpg
---

The crappy firmware has bothered me for some time now. The biggest problem is that it needs additional PC-side software just to receive basic button presses.  
Right now, the only job of the dongle is relaying the messages from my volume knob to my Pi, using mpc to control the music player. Luckily I don't actually need to write my own software and drivers if I just switch to using the HID protocol (and the triggerhappy daemon to assign commands to the hotkeys). This also means that the device works under windows without installing drivers.  

USB is hard enough to implement on its own and the HID protocol is a whole beast in and of itself. I tried writing my own report descriptor more than ten times, but I never managed to get it to work as I want. This meant that I had to find someone elses descriptor. There are many pre-made descriptors available for keyboards and mice, but I quickly found out that a normal keyboard does not support full playback control (volume, next, previous and play/pause). There is, however, another class of HID device available that supports all of these buttons and more: the "Consumer Device". I was luck enough to find [Nick's HID library](https://github.com/NicoHood/HID) which included descriptors and keycodes. Using this descriptor, the dongle worked right out of the box.  

This only left the radio section to be done, but as it only has to receive a single type of packet, this was trivial to implement (at least judging by difficulty. I needed quite some time to find all tiny errors that made the NRF24L01+ do nothing,which is pretty hard to debug).

The new firmware for the dongle and the volume knob [can be found on Github](https://github.com/martin2250/nrf24hid) along with the triggerhappy configuration file for mpc.