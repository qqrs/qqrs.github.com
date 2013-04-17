---
layout: post
title: "Voice- and SMS-Enabled Light Sensor using Raspberry Pi and Twilio"
date: 2013-04-10 12:59
comments: true
categories: hacker-school electronics code
---

Hacker School is three-month self-directed environment for [becoming a better programmer](http://www.hackerschool.com/about) with others doing the same.
I'm in the current batch.  

Several of the current Hacker Schoolers had expressed a desire to learn more about hardware hacking.
[@leocadiotine](http://twitter.com/leocadiotine) and I decided to spend an evening building a Raspberry Pi sensor interface project to show how quick and easy it can be to get into hardware.

[@sashalaundy](http://twitter.com/sashalaundy) gave a short talk this week on what a well-designed web service API should look like.
She used the Twilio API for phone and SMS as an example --- Leo and I decided to give it a try for this project.

## Overview

Hardware and code have the potential to make the world a better place and Leo and I wanted to build a project that improved our surroundings in some small way. 
The Hacker School space in NYC has two restrooms: one attached to the main work area, and one downstairs. 
It's a short walk, but we thought it would nice to know if the bathroom is occupied before taking the time to walk.

{% img right /images/twilio_light_sensor_text_message.png 'Text message response' %}
Our project makes it possible to check the bathroom status by phone or text message.

Bathroom occupancy status is determined using a light sensor attached to a Raspberry Pi. 
If the lights are on in the bathroom, we assume that the bathroom is occupied.

We created a web application hosted on Heroku which accepts periodic bathroom state updates from the Raspberry Pi and handles incoming requests from Twilio. 
When a user calls or texts the Twilio phone number, Twilio sends a request to the web app, which responds with an appropriate message to be spoken or texted to the user.

In addition to the voice and text interface, [@gelstudios](http://twitter.com/gelstudios) created a nice web interface using Bootstrap.

## Server

### Twilio  
Twilio is a web-based service for sending and receiving phone calls and SMS text messages.
It provides an easy-to-use API accessible via HTTP and a convenient Python package.
A free trial of the service is available, though it does insert a nag notice into outgoing messages.
We used the [Twilio Python Quickstart Tutorials](http://www.twilio.com/docs/quickstart/python) as our introduction.

### Heroku  
Heroku is a service which provides a complete, integrated stack for hosting web applications with a range of choices in language, framework, web server and data store.
We created the server application for the project in Python using the Flask microframework. 
The Heroku Dev Center article [Getting Started with Python on Heroku](http://devcenter.heroku.com/articles/python) is a good walkthrough for setting up Flask on Heroku.

### Server Code  
Full source for the web application can be found at [github.com/qqrs/twilio-light-sensor-server/blob/master/run.py](https://github.com/qqrs/twilio-light-sensor-server/blob/master/run.py).
Interesting sections are included below.

The sensor state is not persisted to database --- 
if the Heroku app instance spins down, the sensor state will be unknown when the next request causes it to spin up again.
For purposes of a demo, the server will be kept active by frequent updates from the remote sensor --- 
it should only spin down when there are no remote sensors sending data.

The `/twilio/voice` and `/twilio/text` routes handle requests from Twilio. 
When a user calls or sends an SMS message to the phone number assigned to our account, Twilio is configured so that it will make an HTTP POST request to these routes.
When the server receives the request from Twilio, it generates an appropriate message indicating the status of the bathroom. 
The message is returned to Twilio in the HTTP response and is sent to the user as either audio (by text-to-speech) or as an SMS message.

The `/update` route accepts sensor state updates from the remote sensor via HTTP POST. 
Each request includes `sensor_id` and `sensor_val` parameters to identify
the sensor and report the current value. 

``` python Server https://github.com/qqrs/twilio-light-sensor-server/blob/master/run.py Github
from flask import Flask, request, render_template, redirect
import time
import twilio.twiml

app = Flask(__name__)

sensor_states = {}
allowed_sensor_ids = [u'upstairs-wc', u'downstairs-wc', u'sidestairs-wc']
allowed_sensor_vals = [u'0', u'1']

@app.route("/twilio/voice", methods=['POST'])
def twilio_voice():
    """Respond to incoming Twilio voice phone requests."""
    resp = twilio.twiml.Response()
    resp.say(get_sensor_state_msg('upstairs-wc'))
    return str(resp)

@app.route("/twilio/text", methods=['POST'])
def twilio_text():
    """Respond to incoming Twilio SMS text message requests."""
    resp = twilio.twiml.Response()
    resp.sms(get_sensor_state_msg('upstairs-wc'))
    return str(resp)

@app.route("/update", methods=['POST'])
def update_state():
    """Update state following request from remote sensor."""
    if 'sensor_id' not in request.form or 'sensor_val' not in request.form:
        return ""

    sensor_id = request.form['sensor_id']
    if sensor_id not in allowed_sensor_ids:
        return ""

    sensor_val = request.form['sensor_val']
    if sensor_val not in allowed_sensor_vals:
        return ""

    sensor_time = int(time.time())
    global sensor_states
    sensor_states[sensor_id] = {'status': sensor_val, 'updated': sensor_time}
    return  ""

@app.route("/", methods=['GET'])
def web_state():
    global sensor_states
    now = int(time.time())
    return render_template('index.html', sensors=sensor_states, time=now)

def get_sensor_state_msg(sensor_id):
    global sensor_states
    sensor = sensor_states[sensor_id]
    state = sensor.get('status')

    if state == '0':
        return 'The bathroom is vacant.'
    elif state == '1':
        return 'The bathroom is occupied.'
    else:
        return 'The bathroom is undefined.'
```


## Remote Sensor  
[{% img right /images/twilio_light_sensor_rpi_hardware.jpg Raspberry Pi with ADC daughterboard and CdS photocell %}](/images/twilio_light_sensor_rpi_hardware.jpg)

### Raspberry Pi  
The [Raspberry Pi](http://www.raspberrypi.org/) is a single-board computer with an ARM-core processor, SD card slot, HDMI and composite video, USB, and optional Ethernet.
While similar ARM dev boards and single-board computers have 
[existed](http://beagleboard.org/)
for [years](http://www.embeddedarm.com/products/arm-sbc.php) 
the Raspberry Pi was able to hit an enticing price point and combines several attractive qualities in one package, 
making it a popular choice for hobby electronics projects.

1. __Price point:__ 
$35 with Ethernet, $25 without (plus cost of shipping, tax, power supply, SD card). 
This is down into Arduino territory --- low enough to build a project without worrying about tearing it apart right away to recover the Raspberry Pi.

1. __Integrated Peripherals:__
It has HDMI video (1080p with hardware decoding), audio, Ethernet, USB (with support for Wi-Fi). 
It also has an expansion header with general purpose input/output (GPIO) pins for interfacing to digital signals (LEDs, pushbuttons) and 
a UART, a SPI bus, and an I²C bus (memory devices, sensors).
The combination of video and networking with low-level interfaces in one low-cost device opens up many possibilities.

1. __Linux OS:__
The 700 MHz ARM11-core processor is fast enough to run a full Linux distribution.
The Raspberry Pi Foundation provides images of Debian and Arch Linux tailored for the hardware.
Many other distributions have been [contributed by users](http://elinux.org/RPi_Distributions).
Having a full Linux OS allows the use of higher-level languages like Python and interactive debugging on the device.

A 5V USB power supply and an SD card with an operating system installed are required to begin using the Raspberry Pi. 
A good guide to getting started can be found at the [elinux.org wiki](http://elinux.org/RPi_Beginners).
Preloaded SD cards can be purchased from several vendors or an operating system image can be [loaded onto a blank SD card](http://elinux.org/RPi_Easy_SD_Card_Setup).

Note that it is important to have a stable power supply. Some USB 5V supplies may be inadequate. 
I experienced SD card corruption several times until I bought a [good supply](http://www.amazon.com/dp/B008R97TOQ) 
and set `over_voltage=2` in the config.txt file as described [here](http://raspberrypi.stackexchange.com/questions/2069/filesystem-corruption-on-the-sd-card).
Any good supply with at leat a 1.0 amp current rating should be acceptable.

[{% img right /images/twilio_light_sensor_photocell_sch.jpg 139 Photocell schematic %}](/images/twilio_light_sensor_photocell_sch.jpg)

### Light Sensor  
The light sensor is a 10k CdS photocell, interfaced to a Raspberry Pi with an analog-to-digital converter (ADC) daughterboard. The light sensor is connected to an input of the ADC in a voltage divider configuration with a 10k resistor. With illumination from overhead lighting, the resistance of the photocell drops to about 1.5k.

### ADC Daughterboard
{% img right /images/twilio_light_sensor_adc_daughterboard.jpg ADC daughterboard %}
An analog-to-digital converter (ADC) is required to read the signal from the light sensor. 
The Raspberry Pi does not have an integrated ADC like the Arduino and many microcontroller dev boards. 
However, it is straightforward to interface an external ADC via the SPI or I²C buses. 
Adafruit provides a good guide to using the Microchip MCP3008: 
[Analog Inputs for Raspberry Pi Using the MCP3008](http://learn.adafruit.com/reading-a-analog-in-and-controlling-audio-volume-with-the-raspberry-pi/overview).

I had soldered up an an MCP3008 on protoboard for another project so I reused it for this project.

A number of assembled [expansion boards](http://elinux.org/RPi_Expansion_Boards) which provide an ADC are available from third-party vendors.

### Raspberry Pi Remote Sensor Monitoring Script
Adafruit provides [sample code](http://learn.adafruit.com/reading-a-analog-in-and-controlling-audio-volume-with-the-raspberry-pi/script) 
in Python to read from the ADC via the SPI bus.

Full source for the remote sensor monitoring script can be found at [github.com/qqrs/twilio-light-sensor-remote/blob/master/twilio_light_sensor.py](https://github.com/qqrs/twilio-light-sensor-remote/blob/master/twilio_light_sensor.py). Interesting sections are included below.

The monitoring script consists of a polling loop which reads the sensor at a defined interval. 
It calls `readadc()`, a function provided by Adafruit which bit-bangs the SPI communication in software --- some versions of the Raspberry Pi did not include hardware SPI.
Sensor state is determined by comparing the ADC value to a fixed threshold --- if greater than the threshold, the light is assumed to be on.
A moving average is used to prevent a noisy read from being interpreted as a change in state. 
The sensor state is reported to the server via an HTTP POST request by `update_server_state()` whenever the sensor state changes, or at least every 60 seconds when it has not changed.

``` python Remote sensor polling loop https://github.com/qqrs/twilio-light-sensor-remote/blob/master/twilio_light_sensor.py Github
SENSOR_ACTIVE_THRESHOLD = 700		# threshold in ADC counts
SENSOR_READ_INTERVAL = 1		# interval in seconds
MAX_SERVER_UPDATE_INTERVAL = 60		# interval in seconds
FILTER_SAMPLES = 5			# samples to average

sensor_hist = list()     # this keeps track of the last potentiometer value
last_state = None
last_update_time = 0

while True:
        # read the analog pin
        sensor_counts = readadc(potentiometer_adc, SPICLK, SPIMOSI, SPIMISO, SPICS)
	if len(sensor_hist) > FILTER_SAMPLES:
		del sensor_hist[0]
        sensor_hist.append(sensor_counts)

	sensor_avg = sum(sensor_hist)/len(sensor_hist)
	sensor_state = sensor_avg > SENSOR_ACTIVE_THRESHOLD

        if DEBUG:
                print ("sensor_counts:", sensor_counts, 
			" sensor_avg:", sensor_avg, 
			" sensor_state: ", sensor_state)


	if (sensor_state != last_state
			or int(time.time()) - last_update_time > MAX_SERVER_UPDATE_INTERVAL):
		if update_server_state(sensor_state):
			last_update_time = int(time.time())	# update was successful
			last_state = sensor_state

        # hang out and do nothing for a half second
        time.sleep(SENSOR_READ_INTERVAL)
```


``` python Remote sensor HTTP POST to server https://github.com/qqrs/twilio-light-sensor-remote/blob/master/twilio_light_sensor.py Github
import urllib

SENSOR_ID = "sensor_name_here"
SERVER_UPDATE_URL = "https://app-name-here.herokuapp.com/update"
def update_server_state(state):
	global SENSOR_ID
	global SERVER_UPDATE_URL

	sensor_val = "1" if state else "0"

	params = {}
	params['sensor_id'] = SENSOR_ID
	params['sensor_val'] = sensor_val
	params = urllib.urlencode(params)

	try:
		f = urllib.urlopen(SERVER_UPDATE_URL, params)
	except IOError as e:
		if DEBUG:
			print "I/O error %s: %s" % (e.errno, e.strerror)
			print "Connection error on HTTP POST to %s" % SERVER_UPDATE_URL
		return False

	return f.getcode() == 200
```

To run the script, Python packages must be installed. See the Adafruit article 
[here](http://learn.adafruit.com/reading-a-analog-in-and-controlling-audio-volume-with-the-raspberry-pi/necessary-packages)
on installing `python-dev` and `rpi.gpio`.
The script can be run at the terminal: `python twilio_light_sensor.py`. 

To run the script automatically at boot, it can be added to the end of `/etc/rc.local`, 
immediately before the `exit 0` line:
`python /home/pi/twilio_light_sensor/twilio_light_sensor.py &`
(modifying the path to script if necessary). 

Note that the `&` is required to allow the rest of the init sequence to complete.
The script can be stopped by switching to another tty with Ctrl-Alt-F2, logging in, finding the process ID with `ps aux | grep twilio`, and killing the process with `kill <pid>`.

## Success
With the server running on Heroku and the light sensor attached to the Raspberry Pi, Twilio responds to text messages with the bathroom status: `The bathroom is vacant.` or `The bathroom is occupied.`. Same for voice calls --- it speaks the bathroom status. Success!
