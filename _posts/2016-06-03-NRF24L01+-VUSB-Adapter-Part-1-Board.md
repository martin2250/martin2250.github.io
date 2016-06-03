---
layout: post
title: V-USB to NRF24L01+ Adapter Part 1 - Board
bigimg: /img/nrfVUSB_BoardR1_eagle.png
---

I've decided against using WIFI for my Volume knob, instead I'll use the much more energy efficient NRF24L01+ which you can get cheaply from the usual sources.
That's why I've designed this board:  

![PCB](/img/nrfVUSB_BoardR1_oshpark.png)  

It sports an AtTiny 861 and the bare minimum of external components to interface to USB and the NRF (except for the two diodes on the USB data lines which aren't strictly necessary).
The NRF24L01+ usually comes on breaout boards with a surprisingly consistent and widespread pinout, so you can easily swap out the basic one for an advanced version with an RF PA for longer range.  

**But wait a minute!** USB on an AtTiny? - Yes. V-USB is a complete software (aka bitbanging) implementation of USB 1.1 that runs on AVRs with as little as 2k of ROM. I've used it before and it ran very stable in my testing.  

I'll also write a linux driver for the board, along with a Python library.
If I have time I'll also write an alternative firmware (or even just an alternative operating mode) which will allow you to use the board as a SPI / I²C / UART / GPIO interface.  

Since I've tried to keep the board as small as possible, there is no dedicated AVR-ISP connector, so to flash the 'tiny you'll have to use the SPI lines from the NRF24 connector and the reset line from the GPIO header.  
The board is only 1.3 by 0.7 inches, totalling just $4.65 from [OSHPark](http://www.oshpark.com). I hope it won't be too hard to hand-solder. If it proves too difficult, at least I'll have a reason to try hot air for the first time.

BOM:

* [PCB](https://oshpark.com/shared_projects/8y6aOAnr)
* TS 1117 BCW33LDO  
* ATTINY 861A-SU  
* BAS 40-04 SMD  (x2)  
* X7R-G1206 1,0/50  
* MPE 094-2-008  
* USB AWF  
* 4 resistors, not included in total  
  
Total: 3.01€ from [Reichelt.de](http://www.reichelt.de)
