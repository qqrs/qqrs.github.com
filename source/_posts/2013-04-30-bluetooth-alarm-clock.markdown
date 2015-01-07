---
layout: post
title: "Android-Controlled Bluetooth Double-Bell Alarm Clock"
date: 2013-04-30 17:29
comments: false
categories: code electronics hacker-school
---

{% img center /images/btalarm/btalarm_phone_with_alarm.jpg Alarm clock with phone %}

<!--
Careful design is important when it comes to auditory cues, just as it is for visual elements.
Because the connections we make with sounds can have a highly emotional character, auditory cues
work to amplify our experiences.

Some pleasant examples:
A TV that chirrups a "hello" tune when it turns on.
A clothes dryer that rumbles quietly and then respectfully dings a single time to indicate that the cycle is complete.

Some not-so-pleasant examples:
A burglar alarm keypad that plays a confirmation beep slightly out-of-sync with a button press.
A microwave that impatiently beeps that it's finished every minute even though your hands are full elsewhere in the kitchen.
-->

I've woken many mornings to a blaring square-wave-through-overdriven-8Ω-speaker alarm tone, wailing for me to come back from the comfortable, warm depths of sleep...

> If I had to describe the character of an ideal alarm clock, I would say that it should be **persistent** but **polite**.

As an exercise in curiosity, I bought an analog-face alarm clock with a double-bell ringer and found the alarm sound to be crisp and clear.
However, the clock wasn't perfect:
the second hand tick-tocked all night long, the alarm ringer was too insistent, and I had to remember to set the alarm every evening.

In search of an improved alarm clock, I decided to modify the clock so it could be controlled by my Android phone.
I stripped out the clock mechanism and added a Bluetooth module, then wrote an Android app to gently ding the alarm ringer when the alarm clock on the phone goes off.

It may not be the perfect alarm clock, but I've been very happy with it so far.

## Hardware

### Alarm Clock
{% img center /images/btalarm/btalarm_alarm_interior_sm.jpg Alarm clock interior %}
I used an alarm clock with double-bell mechanical ringer (cost: about $10).
The clock hands are advanced by what appears to be a stepper motor winding built into the mechanism, which is driven by an epoxy-potted timing circuit.
The clock is set by turning knobs to move the hour, minute, and alarm hands.

When the alarm time is reached, an electric circuit is completed by mechanical contact, which turns on a small DC motor to strike the alarm bells---no electronics here!
The DC motor runs from two 1.5V AA batteries in parallel.

### Bluetooth Module
{% img center /images/btalarm/btalarm_rn41_sm.jpg RN-41 Bluetooth module %}

For the Bluetooth interface, I used a
[Roving Networks RN-41](http://www.rovingnetworks.com/products/RN41)
Bluetooth module
(update 1/3/2015: if I was picking a Bluetooth module today, I'd take a look at the [Octopart Common Parts Library](http://octopart.com/common-parts-library) first).

The module is a fully integrated solution which provides everything needed for a Bluetooth Serial Port Profile (SPP) link.
Bluetooth SPP emulates a serial port: data sent over the Bluetooth link is passed through to the UART pins.
The module also provides several general purpose I/O (GPIO) pins which are controlled by entering a command mode.
The GPIO functionality was exactly what I needed for this project.

For a previous project, I had designed a custom PCB for the RN-41 that I was able to re-use as a breakout board.
GPIO 3 was the most accessible on my PCB so I used it to control the ringer motor.

The RN-41 requires a 3.3V supply.
I had a 5V wall adapter handy so I used it along with a 3.3V LDO regulator to power the RN-41.

### RN-41 Command Mode
The Roving Networks [Advanced User Manual](http://www.rovingnetworks.com/resources/download/47/Advanced_User_Manual)
(update 1/3/2015: unfortunately the page seems to have been removed)
documents the command set for controlling the Bluetooth module.
Sending `$$$` causes the module to enter command mode where it will interpret received data as configuration commands rather than passing the data through to the UART.
The module stays in command mode until it sees the command `---`.

Two things to note: every command except `$$$` is followed by a carriage return character, and,
by default, it is only possible to enter command mode within 60 seconds of cycling power.

To test the Bluetooth link, I used the [Bluetooth SPP](https://play.google.com/store/apps/details?id=mobi.dzs.android.BluetoothSPP) Android app.
After pairing with the RN-41 Bluetooth module, I used the Bluetooth SPP app to confirm that it responded to commands as expected,
then modified some configuration settings.

Here are the commands I used:

```
# one-time configuration -- saved to flash memory
SN,btalarm      # set the Bluetooth device name
ST,255          # enter command mode at any time (no 60 second window after boot)
SQ,4            # disable special functions on GPIO 3 and 6

# GPIO control
S&,0808         # mask and turn on GPIO 3
S&,0800         # mask and turn off GPIO 3
```

### Motor Switching
{% img center /images/btalarm/btalarm_sch.png Bluetooth alarm schematic %}
{% img right /images/btalarm/btalarm_fet_protoboard_sm.jpg Bluetooth alarm %}
First, I soldered a 1N4004 diode at the motor terminals for flyback suppression.

I didn't have any test gear handy to see how much current the motor draws.
Since it was powered by two AA batteries in parallel, it seemed safe to assume that it could easily require up to a couple hundred milliamps.

I tried using a MPS2222A NPN transistor to switch the motor.
I found that the motor turned on when forcing the base high through a resistor to +3.3V but not when controlled by the RN-41 GPIO pin.
I suspect the RN-41 GPIO pin was unable to source enough current to drive the transistor into saturation.
However, the datasheet didn't specify a current limit for the pins and without test gear it wasn't worth speculating.

I swapped the MPS2222A for an International Rectifier n-channel power FET with 3.3V-tolerant gate threshold.
Success! The Bluetooth module turned on the ringer motor.



## Android application

[{% img right /images/btalarm/btalarm_screenshot_select_device_sm.png Bluetooth alarm %}](/images/btalarm/full_size/btalarm_screenshot_select_device.png)
This was my first time creating an Android app so I started out by
[installing the SDK](http://developer.android.com/sdk/index.html)
and working through the
[Building Your First App](http://developer.android.com/training/basics/firstapp/index.html)
example in the Android developer docs.
The whole process turned out to be very easy---I&nbsp;was running a "Hello, World" app on my phone within a couple of hours.
Quite a contrast to the development tools for some of the microcontroller and FPGA platforms I've used in the past!

For the most part, I tested using an actual phone.
While it's nice that the Android emulator is available, I found it to be very sluggish.

The Android SDK comes with a Bluetooth Chat sample project, which demonstrates how to create a background service in a separate thread to handle Bluetooth communications.
To communicate with the RN-41 module, I found that I needed to change the Bluetooth UUID to the
[proper value for an SPP connection](http://stackoverflow.com/questions/3072776/android-bluetooth-cant-connect-out).
I converted the chat app into a debug terminal, allowing me to send arbitrary commands to the RN-41 or press shortcut buttons for command mode and GPIO control.

[{% img left /images/btalarm/btalarm_screenshot_debug_sm.png Bluetooth alarm %}](/images/btalarm/full_size/btalarm_screenshot_debug.png)
I ran into an error when trying to run the Android example:
`getActionBar() call returning null`.
A StackOverflow search provided a fix which involved a
[change to the *AndroidManifest.xml* file](http://stackoverflow.com/questions/6867076/getactionbar-returns-null)
for the project.

I thought it would be elegant to trigger the Bluetooth alarm clock from the built-in Nexus 4 Android 4.2 alarm clock so,
ideally, I wanted to find a way to intercept alarm events from the system alarm clock.
I [found a post](https://groups.google.com/forum/?fromgroups=#!topic/android-developers/QTpdFvtdRVo) that pointed me in the right direction.
The alarm event can be captured by creating a BroadcastReceiver to receive the `com.android.deskclock.ALARM_ALERT` action.
Note that alarm events from other alarm clocks (older Android builds, manufacturer/carrier builds, third party apps) will use
[different actions](http://stackoverflow.com/a/15876187).

[{% img right /images/btalarm/btalarm_screenshot_settings_sm.png Bluetooth alarm %}](/images/btalarm/full_size/btalarm_screenshot_settings.png)
My alarm app is implemented using a background service that registers the BroadcastReceiver to listen for alarm events and starts the ringer when an event occurs.
The service is loaded when the app is installed or the phone is powered up.
Since I wanted to support a "repeated ding" ringer style, I also created a separate alarm ringer background service thread
to support sending a timed pattern of Bluetooth commands, with pauses.

I wanted to be able to set configuration options to enable/disable the alarm, change the ringer style, and save the selected Bluetooth device
so I created the appropriate UI controls and used the Android SharedPreferences API to store the settings.

Full source for the Android application can be found at [github.com/qqrs/btalarm](https://github.com/qqrs/btalarm).


### Acknowledgments

This was a project I worked on at [Hacker School](http://www.hackerschool.com).
Hacker School is a free three-month self-directed learning environment
that has been described as being "like a writers' retreat for programmers."

Many thanks to [@leocadiotine](http://twitter.com/leocadiotine) for repeatedly offering expert guidance in navigating the intricacies of the Android APIs.
