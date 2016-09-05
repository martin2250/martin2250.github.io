---
layout: page
title: NRF24L01 V-USB dongle
bigimg: /img/nrf24vusb_poster.jpg
---

NRF24L01+ V-USB dongle
==

This project statrted out as just a part of my wireless volume knob. I needed a way to connect an AVR to a Raspberry Pi wirelessly, running from battery on the transmit side.  
This pretty much ruled out using an ESP8266 over wifi. Apart from using too much power, it is also slow to start up which would result in an unacceptable delay.  
The NRF24L01+ on the other hand is pretty much perfect for this job. It handles encoding, CRC, retransmission and addressing while using mere nanoamperes in power-down mode and starting up in a matter of milliseconds.

You could of course interface it directly with a Raspberry Pi, but as I plan to switch my sound system to my Odroid C1 soon, I wanted a more portable solution. Having used V-USB before, the decision was a pretty easy one.  
This also has the benefit of offloading much of the work from the SBC, and of course it also works on normal desktops.

Nordic Semiconductor, the manufacturer of the NRF24 series of chips also produces their own line of microcontrollers with integrated 2.4GHz transceiver and even an actual USB interface like the [nRF24LU1+](http://www.nordicsemi.com/eng/Products/2.4GHz-RF/nRF24LU1P). These chips often get used in wireless mouse/keyboard dongles and would fit this project perfectly as they are readily available with fully populated PCBS and enclosures.
Unfortunately, there are no cheap solutions for programming these chips and I don't have any experience working with 8051s, so I chose to stick with AVRs.

Hardware
--------

V-USB requires only two pins which both share the same port and one of which must be an interrupt pin. Of all ATTiny's, the 'Tiny861 fits this requirement perfectly as the port which INT0 is on also has exactly one more pin left that is not used for the crystal, SPI or reset.

The hardware design was pretty straightforward. Save for the voltage regulator, there are only some passives. I also added some BAS 40-04's to clamp the USB lines to the supply voltage just for good measure. To save some space, I also decided against adding a separate ICP programming header, so I made an adapter to combiney the SPI pins from the female header and reset to a standard 6-pin AVR-ISP header.
My PCB measures 18x34mm ,is populated on both sides and can easily be hand-soldered. I ordered a pack of three from OSHPark and promptly assembled one.
![](/img/vusb-nrf24-01.jpg)
After flashing a basic VUSB demo, I was greeted by the familiar usb-device-recognized-chime. This part of the hardware works. The boards have just a single error: MOSI and MISO are swapped. I didn't consider that the USI found in many AtTinys does not work like a proper hardware SPI. Since I noticed this before assembling the first board, it was pretty easy to cut the traces and fix them with some enamelled copper wire. The PCB layout on GitHub does no longer have this mistake.

HID firmware
--------

This is the firmware that I'm using currently. It replaces the ugly custom-protocol + crappy daemon system. It enumerates as a "Consumer device" ( = fancy keyboard), only receives messages from my volume knob and sends them to the host as volume up/down, next/previous song and pla/pause keystrokes.  
You can find the current version on [Github](https://github.com/martin2250/nrf24hid). It is definitely worth a look, especially if you want to implement your own HID device.

Generic (Debug) firmware
--------

The first iteration of the software exposed individual registers of the NRF24L01+ to the PC, later I decided to let the AtTiny handle all communication with the transceiver. Right now, the protocol consists of just 4 different messages:

|Direction		|bRequest	|size			|function	|
|---------------|-----------|---------------|-----------|
|Host -> Dev	|0x01		|16				|config		|
|Host -> Dev	|0x02		|0				|reset		|
|Dev -> Host	|0x03		|1				|status		|
|Host -> Dev	|0x10		|6 + payload	|TX packet	|
|Dev -> Host	|0x20		|2 + payload	|RX packet	|

The exact structure of the packets is documented in "Protocol.xlsx" in the Github repo.

The microcontroller has it's own internal buffer where one received packet is stored (if available). This way it can immediately reply to a USB request (SPI would be more than fast enough to get this data on the fly but it's simpler that way). This brings the total amount of buffered packets to four (the NRF24L01+ has an internal 3-packet FIFO).

TX packets also have one buffer spot on the microcontroller, it will automatically switch to PTX mode and back when neccessary. The destination addresses are also handled automatically. If the TX address has changed from the last transmission, it will be stored in the TX_ADDR and RX_ADDR_P0 registers of the NRF24L01+. If there is an ongoing transmission on the NRF24L01+ and the destinations don't match, the packet will wait in the buffer until the transmissions to the previous destinations are finished.

To make the code simpler, I decided to dedicate pipe 0 of the NRF24L01+ entirely to receiving ack packets. This means that only P1 through 5 are available for receiving packets. Be aware that all five pipes share the first four bytes of the address, so only the LSB can differ.  
Also if you want to build your own client, remember to transfer the five address bytes in reverse order (LSB first).

PC Software
-----------

I also wrote a simple GUI for debugging, [you can find it on GitHub](https://github.com/martin2250/NRF24Debug). It is written in C# and uses LibUsbDotNet. It should also run on Mono, but I haven't tried that yet.
![](/img/nrf-debug-screenshot.png)
I also plan to write a daemon for linux in C, which will run user-defined scripts depending on the pipe when it rececives a packet. Right now the daemon that acts as receiver for my [Volume Knob](coming soon) is written poorly and specced down a lot (not much power left over on an original Raspberry Pi A running [Volumio](https://volumio.org/)).