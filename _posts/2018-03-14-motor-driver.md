---
layout: post
title: motor drivers for my CNC machine
bigimg: /img/motor-driver-1.jpg
---
My new CNC machine uses JMC's iHSV57-36-18-36 integrated servo motors. One big advantage is that they don't require any external electronics, the encoder, controller and power electronics are all included. They only need power (20-50V) and step/direction signals, the latter of which are galvanically isolated by optocouplers. There is also an RS232 configuration port for setting the parameters for the PID control loops.

#### disclaimer (not at the top so it doesn't show up in the preview) the events outlined in the next few posts on my cnc machine all happened three to five months ago. I won't write these in chronological order (or in any sensible order at all) but I will adjust the publish dates so they will apppear in chronological order.

The step/direction inputs have internal constant current sources.
![todo insert I(U)-graph]()
They reach their nominal current at about 3V, though the manufacturer recommends at least 5V. I ultimately want to use either an STM32 microcontroller or a beaglebone black to generate step signals, both of which use 3.3V. I did some testing with an Arduino Due (also 3.3V), which seemed to work fine without a level translator.
Nonetheless I wanted to stay in the safe side, so I built some level converters to boost the voltage levels and provide some protection from ESD.

Since the servos can be configured in both, common anode and common cathode, modes, I opted for a single mosfet circuit, with the gate protected by zener diodes. The circuit also handles the servo enable input and both outputs: 'at position' and alarm. The 'at position' output indicates if the actual position matches the setpoint and is only used to light up an LED. The alarm output indicates an alarm state (eg. overtemperature, overcurrent, position error threshold reached). It is also fed to the main board via a open-drain output. As both the anode and cathode of the alarm output optocoupler are accessible, the transistor is not strictly neccessary, but I included it anyways for ESD protection and to provide enough current for an indicator LED. This LED is separated from the interface with an additional diode (D6) so only the LED for the failed motor lights up. The driver board is connected to the main controller board via a six-pin header that carries the signals, the 12V rail and ground.
[![schematic](/img/CNC/motor-driver/schematic.png)](/img/CNC/motor-driver/schematic.pdf)

The layout is fairly tight and is quite difficult to solder, especially without a solder mask.
![board layout](/img/CNC/motor-driver/board.png)

I always wanted to try making double sided PCBs and with my new cnc machine already working I tried it for the first time. To align the board for milling the second side, I used four pins cut from paperclips. FlatCAM was used to generate the toolpath and handle the alignment of top and bottom side. The actual milling process went very well, though it took me well over half an hour for every board. The alignment between top and bottom layer is nearly perfect, the 0.6mm vias line up perfectly on both sides.
![top side](/img/CNC/motor-driver/board-top.jpg)
![bottom side](/img/CNC/motor-driver/board-bot.jpg)

Assembling the boards was not too hard, though it was a long and painstaking process. The finished boards were covered in clear nail varnish to protect them from oxidation and contaminatio. Here are the four finished motor drivers along with some other boards for my CNC:
![board pile](/img/CNC/motor-driver/board-pile.jpg)
