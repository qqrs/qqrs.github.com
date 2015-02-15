---
published: false
layout: post
title: "Vim on a Mechanical Typewriter"
date: 2013-05-03
comments: true
categories: code electronics hacker-school
---

<iframe src="http://gfycat.com/ifr/BackGiddyAmericanbittern" frameborder="0" scrolling="no" width="720" height="540" style="-webkit-backface-visibility: hidden;-webkit-transform: scale(1);position: relative;left: -32px;" ></iframe>

## Inspiration

I've had the idea for this project kicking around for years, but it was only recently that a minimal-assembly-required interfacing scheme presented itself and I found the time to make it happen.

The original inspiration came a long time ago when I read a writeup by someone
who had thoughtfully interfaced a typewriter to a computer so an older relative who wasn't comfortable using a computer could send email.
I seem to recall that it was an IBM Selectric or similar electric typewriter with serial out.
(It may be [this project](http://hackaday.com/2007/07/25/emailing-typewriter/), but the linked project page has disappeared.)
I thought the idea was fun, but I wanted to use a mechanical typewriter.

At some point, the [USB typewriter](http://www.usbtypewriter.com/) project popped up which does exactly that.
The [technical writeup](http://www.usbtypewriter.com/pages/instructions) indicates that
it uses a long PCB strip with a laser-cut metal tab for each key.
The tabs are folded over the crossbar on the underside of the typewriter to detect strikes of the key levers on the crossbar.
To detect key strikes, an ATmega168 with a bunch of 74HC595 shift registers strobes the tabs and tests for voltage at a common point shared by the key levers.
It looks like a nicely polished project, and it's great to see the creator running what appears to be a successful Etsy store while supporting open hardware.

That approach required more assembly work than I wanted to put in, though, so my idea was on hold until I stumbled across the SoftPot while browsing SparkFun.

## SoftPot

{% img center /images/typewriter/softpot.jpg SoftPot %}
The SoftPot is a touch-sensitive position sensor produced by [Spectra Symbol](http://www.spectrasymbol.com/potentiometer/softpot).
I've written a [separate post](/blog/2013/04/22/interfacing-a-softpot-sensor-to-an-adc/)
describing the SoftPot and documenting how to interface it to a microcontroller ADC,
but for most uses it can be thought of as a momentary-contact potentiometer and can be modeled as a voltage divider.
When touched, the output voltage is proportional to the position of a touch along the length of the sensor.

I installed the SoftPot (a [ThinPot TSP-L-0200-103-1%-RH](https://octopart.com/tsp-l-0200-103-1%25-rh-spectra+symbol-19249699)) using its adhesive backing onto the upper side of the crossbar.
The purpose of the crossbar is to advance the carriage by one space after the type hammer leaves its imprint on the page.
To accomplish this, the lever arm for each key applies force to the top of the crossbar when the key is pressed.
Since the key levers are arranged along the length of the crossbar, the position of a hit on the crossbar can be used to determine which key was pressed.

There are a few keys which don't actuate the crossbar and therefore can't be detected by the SoftPot: space bar, backspace, shift, and tab.

{% img center /images/typewriter/typewriter_underside.jpg Typewriter underside %}

I did run into some difficulties mounting the SoftPot securely while still allowing the crossbar to move.
I ended up running the SoftPot connector and wires up around a piece of the typewriter frame, down around another piece, then outward along the underside of the typewriter.
For strain relief I added some electrical tape to secure everything, but I'm still worried that repeated stresses at the connector will eventually damage the SoftPot.
For a more robust build, I would probably either need a SoftPot with a right-angle connector (which I don't believe is available), or I would need to take a Dremel to the crossbar side support.

There is also a small inactive zone on the far edge of the SoftPot from the connector, so the `q` and `=` keys aren't detected at all.
Mounting the SoftPot with the end curving up onto the crossbar side support might fix this, but it would be at greater risk of slipping or being damaged.
It might be possible to trim the SoftPot by a few millimeters,
but it's difficult to tell from the datasheet if that would interfere with its operation or expose the internals to air that might degrade it prematurely.

Overall, I'm quite satisfied with the performance of the SoftPot given the limited ambitions of this project.
It's able to consistently detect keypresses and seems to have no trouble reliably distingushing between adjacent keys.
With some practice, it could easily be installed on a new typewriter in just a few minutes.

## Raspberry Pi
{% img right /images/twilio_light_sensor_adc_daughterboard.jpg ADC daughterboard %}
I had a Raspbery Pi with Microchip MCP3008 ADC daughterboard handy (described in more detail in a [separate blog post](/blog/2013/04/10/twilio-light-sensor/))
so I used it to take the SoftPot sensor readings.

I added a 100kΩ pulldown resistor to the SoftPot output to prevent the ADC input from floating when no keys are pressed.

### Software
The software to read the SoftPot sensor values and convert to keystrokes is written in Python. Source code is here:
[github.com/qqrs/ttypewriter](https://github.com/qqrs/ttypewriter).

The implementation is rough and leaves plenty of room for future improvement, but for purposes of testing out the SoftPot sensor it got the job done.

The script consists of a polling loop that reads the sensor at a fixed delay.
To read values from the MCP3008 ADC, it calls `adc_spi.py`, which is based on
[example code](http://learn.adafruit.com/reading-a-analog-in-and-controlling-audio-volume-with-the-raspberry-pi/necessary-packages)
from Adafruit that bit-bangs the SPI communication in software (some versions of the Raspberry Pi did not include hardware SPI).
The script is able to read the sensor at about 100 samples/second.
This is slow but it provided adequate key detection results so I haven't yet gone on to modify it to do the SPI in a separate thread, use the hardware SPI interface, port to C, etc.

Key up/down state is determined by comparing the sensor value to a fixed threshold of a few ADC counts.
When a key is pressed, the sensor value is recorded until all keys are up.
As a debouncing mechanism, very short keypresses of < 100 ms are discarded.
Otherwise, outlier values are filtered out and the average of the sensor readings is taken.
A calibration file maps sensor reading values to typewriter keys.

One major caveat of this approach is that for a key to be detected correctly, it must be fully released before the next key is pressed.
The potentiometer-like behavior of the SoftPot means that there is no simple means of decoding multiple simultaneous pressed keys.
While it would be useful to improve the software to handle faster typing speeds, in practice the typewriter tends to enforce its own speed limit since sustained fast typing is likely to jam the typeheads.

For output, the script supports writing keypresses to stdout or injecting them into a `screen` session.

Since the original `vi` key commands were designed for use over a teletype interface, the `vim` text editor is very usable without any special keys or modifier keys.
With a `vim` instance open inside a `screen` session, `jj` mapped to `Esc`, and the key detection script running, the typewriter is usable for text editing.

## Ideas for Future Improvements

1. Improve the keypress detection algorithm to handle faster typing with overlapping keypresses.
1. Add limit switches or reed switches and magnets to detect presses of the keys not detected by the SoftPot.
1. Port the code to an Arduino or a microcontroller with a USB HID stack so the typewriter can be used with any computer.
1. Add a [vim clutch](https://github.com/alevchuk/vim-clutch).
1. Real life [Write or Die](http://writeordie.com/):
port to MSP430 for low standby power consumption, add an SD card, and add an energy harvesting system (piezo? electromechanical?) to capture energy from keypresses.
If you type fast enough you'll store enough energy to write your work to the SD card before it's lost!
