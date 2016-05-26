---
layout: post
title: Wifi Volume Knob Part 1
bigimg: /img/WifiVolumeKnobPrototype1.jpg
---

**Why?**
I recently started using [Volumio](https://volumio.org/) as my main music player, but I soon got fed up with opening up the webinterface every time I want to change the volume or pause the song.
My inspiration is [this beatiful piece of work](http://runawaybrainz.blogspot.de/2013/11/adafruit-trinket-compatible-volume-usb.html) by Rupert Hirst.

**What?**
I want to build a similar-looking, probably larger knob that works over wifi, ideally running from a Lithium-Ion battery with an integrated USB charger. I also want to include basic playback controls while keeping the exterior elegant.

**Features I want to include:**

* Change the volume by rotating the knob
* Play/pause the current song by pressing the knob
* Change the song by pressing and turning the knob
* Use capacitive sensing to activate the wifi controller before touching the knob to decrease the initial latency.
* Simple WebUI that allows to change the network and volumio IP address

**How?**
I'll use an ESP8266 module for wifi and handling the web interface, it will be accompanied by a low power attiny that handles the capacitive sensing and rotary encoder input so it won't miss any clicks that happen before the ESP is running.

Prototype
-----
![Prototype](/img/WifiVolumeKnobPrototype1.jpg)
I have a breadboard prototype with only the ESP8266 running and so far it's working great. I used the [ESP8266 core for Arduino](https://github.com/esp8266/Arduino) for programming.

I probably won't have time to do much more than prototyping until the semester break in august, but I'm really excited to see how this turns out. Until then I'll probably just use the prototype as it works pretty good so far.
