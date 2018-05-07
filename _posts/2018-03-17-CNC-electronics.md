---
layout: post
title: CNC machine electronics
#bigimg: /img/CNC/input-buffer/input-buffer-title.jpg
---
My new CNC machine didn't arrive as a complete kit. The mechanics came from alufraese.de and the JMC servos were imported straight from china. That leaves all the electronics, which I designed myself.

Since I don't know what controller I ultimately want to use, I designed the electronics to be as modular as possible. This has the benefits of the individual PCBs being less complex and therefore making them easier to make on the CNC, also I could reuse as much of the electronics as possible when I switch to another controller.

### power supply
My CNC machine needs two power supplies: a 48V power supply for the servos and a 12V supply that powers the electronics.
The two power supplies used are the meanwell MW-XXXX and MW-XXXX.

### Boards
#### Motor Driver
Probably the most important boards are level converters that convert the 3.3 to 5V output of the controller to a higher voltage. This makes the connection less susceptible to EMI, reduces chances of damaging the controller via ESD (or at least limit the damage to an individual driver) and moves the bulky screw terminals going to the motors off the main controller board. The driver boards are connected to the controller via a standard six-pin IDE connector and ribbon cable. The cable carries 12V, GND, step, direction, motor enable and a open drain alarm output.

#### Relay Board
The CNC machine needs relays to switch loads at mains voltage, for example the spindle or later some form of dust extraction. This moves the 240V portion of the wiring away from the main board. Apart from the two relays controlled by the controller, there is a third relay, which switches on as soon as there is 12V power. This relay serves to short out the resistor that limits the inrush current of the two power supplies after the main filter capacitors are charged up.
This board is also connected to the 12V power supply and passes the voltage forward to the main board.
The six-pin IDE connector carries the two relay signals as well as 12V and ground. The power is 

#### Input Buffer Board
To connect limit switches and various probes (3D edge finder, tool length sensor or just a plain tool touching a conductive workpiece) I designed a board that has very low impedance inputs (470 ohm resistor to the 12V rail) and provides an open-drain output which can be used with both 5V and 3.3V controllers.
