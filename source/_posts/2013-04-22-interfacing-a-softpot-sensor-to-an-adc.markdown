---
layout: post
title: "Interfacing a SoftPot Membrane Potentiometer"
date: 2013-04-22 12:26
comments: true
categories: electronics
---

### SoftPot?
{% img right /images/softpot_200px_01.jpg 'SoftPot' %}

The SoftPot is a touch--sensitive position sensor produced by [Spectra Symbol](http://www.spectrasymbol.com/potentiometer/softpot).
It is available in a variety of sizes and configurations for linear and angular position measurements.
There are also two related product lines: the ThinPot, which is narrower, and the HotPot, which is rated for higher temperature operation.

A SoftPot acts as a momentary--contact potentiometer. 
When a finger or mechanical stylus presses down on the sensor area, a top conductive shunt layer makes contact with a lower resistive layer.
The top layer serves the same purpose as a movable wiper in a traditional rotary potentiometer, dividing the resistive layer into two segments around the point of contact. 

More detail on the SoftPot can be found at the [NYU ITP Sensor Workshop](http://itp.nyu.edu/physcomp/sensors/Reports/SoftPot)
(update 1/2/2015: unfortunately the page seems to have been removed and is not available from web.archive.org).

Several vendors sell the SoftPot in small quantities, including [SparkFun](http://www.sparkfun.com), [Adafruit](http://www.adafruit.com/), and [Digikey](http://www.digikey.com/).

While reading up on the SoftPot to prepare for using it in a project, I found that a number of people posting in the SparkFun comments had run into problems
since the datasheet is pretty sparse and there are few other resources available.
It's a pretty handy sensor for certain applications and it would be a shame to see people avoiding it due to the limited documentation.

### Interfacing a SoftPot
{% img right /images/softpot_pot_divider.png 'Potentiometer voltage divider' %}
The SoftPot datasheet indicates that it can be interfaced in the same way as a traditional rotary potentiometer.
There are two bus bar pins, which are connected to power (pin 1) and ground (pin 3). 
The datasheet refers to pin 2 as the "collector," which is analogous to the wiper in a rotary potentiometer.
With power and ground connected, the collector voltage is proportional to the position of the touch along the length of the SoftPot.

This configuration works fine as long as the SoftPot is always activated by a touch.
However, it is not immediately clear from the datasheet how the sensor responds when not touched.

I connected my SoftPot (part number SP-L-0300-103-1%-RH) to the ADC input of a microcontroller and found that the ADC recorded a slow oscillation of a few hundred millivolts when the SoftPot was not touched.
This seemed to indicate that the SoftPot leaves the collector floating when not touched, so I took some resistance measurements.

{% img right /images/softpot_connection_diagram.png 'Softpot connection diagram' %}

|                   | Pin 1-3 | Pin 2-3 | Pin 1-2 |
|:------------------|--------:|--------:|--------:| 
| no touch          | 9.8 kΩ | > 10 MΩ | > 10 MΩ | 
| touch far end     | 9.6 kΩ | 20 Ω | 9.6 kΩ | 
| touch middle      | 9.5 kΩ | 4.7 kΩ | 4.7 kΩ | 
| touch close end   | 9.6 kΩ | 9.6 kΩ | 5 Ω | 

<br>
That confirmed it. The collector pin is floating when the sensor is not touched.

Also worth noting is that the resistance from pin 1 to 3 decreases slightly when the SoftPot is touched. When touching with a fingernail instead of a fingertip, 
the resistance decrease is smaller, 
so this effect seems to be caused by the conductive shunt layer shorting out a cross section of the resistive layer at the point of contact.

**Warning:** By pressing both ends of the SoftPot at the same time it is possible to short power to ground, which can damage the SoftPot. 
This may be unlikely with the linear SoftPot, but is a serious concern for the rotary SoftPot since a stray finger could easily press both ends at the same time.
A resistor in series with the supply pin should protect against this, but will reduce the voltage range of the output.

### SoftPot with Pulldown
{% img right /images/softpot_pot_divider_pulldown.png 'Potentiometer voltage divider with pulldown' %}

We need a way to distinguish between a garbage sensor measurement due to the floating collector pin and a good sensor measurement resulting from a touch.
One way to do so is to add a pulldown resistor at the SoftPot collector pin that will hold the ADC input at GND when the SoftPot is not touched.

Adding a pulldown resistor will affect the linearity of the sensor measurement.
The choice of resistance value is a tradeoff: 
it must be small enough to hold the ADC pin near GND but as large as possible to reduce the linearity error introduced in the measurement.

{% img right /images/softpot_res_divider_pulldown.png 'Softpot equivalent circuit with pulldown' %}
The SoftPot can be modeled as two series resistors, 
$$ R_1 $$ and $$ R_2 $$, where  $$ R_1 + R_2 = 10kΩ $$.
The touch position can be denoted as $$ x $$, with $$ x = 0.0 $$ corresponding to the close end of the sensor and $$ x = 1.0 $$ to the far end.
Since the resistances are proportional to touch position, this gives $$ R_1 = x \cdot 10kΩ $$ and $$ R_2 = (1-x) \cdot 10kΩ $$.


When a pulldown resistor $$ R_o $$ is added, it combines in parallel with $$ R_1 $$ 
and the voltage at the ADC input can be calculated as 
$$ V_{out} = \frac{R_1 \| R_o}{(R_1 \| R_o) + R_2}\cdot{V_{cc}} $$,
where $$ R_1 \| R_o = (\frac{1}{R_1} + \frac{1}{R_o})^{-1} $$.

To illustrate the effect of the pulldown resistor on sensor linearity, 
the top plot below shows $$ \frac{V_{out}}{V_{cc}} $$ vs. touch position for four values of $$ R_o $$: 1&nbsp;kΩ, 10&nbsp;kΩ, 100&nbsp;kΩ, and 1&nbsp;MΩ.
The bottom plot shows linearity error, which is the percentage difference from a true linear response due to the effect of the pulldown resistor.
(Python/matplotlib source code [here](http://gist.github.com/qqrs/4107845)).

{% img center /images/softpot_pulldown_error_sm.png 'SoftPot Pulldown Error' %}

The following table shows the maximum linearity error for each pulldown resistance value. 

| $$ R_o $$  | max error (%)  
|-----------:|-----------:|
|  1 kΩ  | 71.4%
| 10 kΩ  | 20.0%
| 100 kΩ  |  2.4%
|  1 MΩ  |  0.2%

<br>
A good rule of thumb for most circuits is to choose a pulldown resistor an order of magnitude larger than the output resistance of the signal source.
It seems to hold in this case:
any value that is at least 100 kΩ is reasonable, depending on the specifics of the application.

### Conclusion
The SoftPot can be treated like a traditional rotary potentiometer, except when the sensor is not touched. 
When not touched, the sensor output at the collector pin is left floating.
Taking this into account, here are three possibilities for using the sensor:

1. Add a pulldown resistor of approximately 100kΩ -- 1MΩ to hold the sensor output at GND when not touched.
1. Design the device such that a stylus or other object is in continuous mechanical contact with the SoftPot.
1. Use some additional means of detecting touch events, such as a limit switch or capacitive touch sensor. Only read the SoftPot output when a touch has been detected.

Finally, this may not apply to most applications, but if there is a possibility the SoftPot could be touched simultaneously at both ends for an extended period of time
it should be protected with a supply--pin series resistor or other current--limiting circuit.

That's it. The SoftPot is a useful sensor with many possibilities for interesting projects!
