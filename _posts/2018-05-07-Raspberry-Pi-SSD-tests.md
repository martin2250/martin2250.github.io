---
layout: post
title: Running A Raspberry Pi on an SSD
#bigimg:
published: false
---

I've been using a Raspberry Pi 3B together with a HifiBerry Amp and the [MoodeAudio distribution](http://moodeaudio.org/) to play music.
Using it just for that seems like a waste of a perfectly good Pi, so I'm adding more services to it.

One service in particular is a time series database for monitoring different metrics around the house (for now the gas meter reading and weather data from the garden).
This requires a more reliable storage than the SD card I'm currently using, which is why I want to use a SATA hard drive connected via USB.
More specifically I'm choosing an SSD for it's lower power consumption and noise levels (the Pi sits right above my bed).
Other benefits to replacing the SD card would be faster boot up and file access times and more write endurance.

The hardware I'm using is a 120GB WD green SATA SSD and a Sabrent USB 3.0 to SSD adapter.
I'll leave the boot partition on a cheap SD card for now.
This shouldn't cause any reliability issues as the boot partition is read-only.

I want to use this as an opportunity to run a few benchmarks comparing the SSD to the old SD card.
Mainly I'm interested in boot time, sequential / random read and write speeds and power consumption with and without disk activity.

The test conditions are as follows:
- Raspberry Pi 3B
- brand new 16GB SanDisk SD card for reference
- 120GB WD green SATA SSD and a Sabrent USB 3.0 to SSD adapter
- new installation of raspbian stretch lite 2018-04-18 with kernel <TODO>
- shunt resistor in the power supply together with an oscilloscope to measure current draw

The commands for testing disk performance are
[here](https://www.binarylane.com.au/support/solutions/articles/1000055889-how-to-benchmark-disk-i-o)
Also I'm testing boot time (time shown in dmesg for XXX message) and the time it takes to start one hundred instances of the hello world web server docker image.




|                              | SanDisk Ultra 16GB   | WD Green 120GB 2.5"  |
| ---------------------------- |:--------------------:|:--------------------:|
| Write Bandwidth (r+w)        | XXX MB/s             | XXX MB/s             |
| Read Bandwidth  (r+w)        | XXX MB/s             | XXX MB/s             |
| Write Bandwidth (wo)         | XXX MB/s             | XXX MB/s             |
| Read Bandwidth  (ro)         | XXX MB/s             | XXX MB/s             |
| Write IOPS (r+w)             | XXX                  | XXX                  |
| Read IOPS  (r+w)             | XXX                  | XXX                  |
| power consumption (idle)     | XXX W                | XXX W                |
| power consumption (writing)  | XXX W                | XXX W                |
| boot time                    | XXX s                | XXX s                |
| create 100 docker containers | XXX s                | XXX s                |
| start 100 docker containers  | XXX s                | XXX s                |
