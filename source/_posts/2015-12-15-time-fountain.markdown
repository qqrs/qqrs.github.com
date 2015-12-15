---
layout: post
title: "Time Fountain"
date: 2015-12-15 21:00
comments: false
categories: electronics
---

{% img center /images/timefountain.jpg 'Time fountain' %}

## Inspiration

The first time I remember seeing a time fountain was a
[blog post by Nate True from 2006](http://cre.ations.net/creation/the-time-fountain)
[(archive.org)](http://web.archive.org/web/20150414033350/http://cre.ations.net/creation/the-time-fountain).

## Operation

The time fountain works by using flashes from UV LEDs to illuminate falling drops of fluorescent dye.
If the drops fall at a consistent rate, the UV LEDs can be flashed at the same frequency as the drops are falling so that the drops appear to be suspended in mid-air. Or, if there is a small frequency differential, it appers that the drops are rising upward.

[This blog post](http://www.eccentricgenius.com/wp/2006/08/10/stopping-time-visually/)
[(archive.org)](http://web.archive.org/web/20070603015657/http://www.eccentricgenius.com/wp/2006/08/10/stopping-time-visually/) provides a more detailed explanation.

## Construction

I used a 555-timer configured for [low-duty-cycle astable operation](http://pcbheaven.com/wikipages/555_Theory/?p=1), with a 100 uF timing capacitor, 100 ohm resistor on the charging path, and 10k ohm pot on the discharging path. This corresponded to roughly 15 ms on-time and 0.5 Hz to 20 Hz frequency. An n-channel FET switches the LEDs.

While it would be nice to detect drops and reset the cycle so phase errors don't accumulate, the effect is still pretty good with a simple timer as long as you have consistent drops.

I bought some fluorescein dye from Ebay but it turns out that highlighter dye works just fine and doesn't precipitate out of solution like the fluorescein. I diluted the dye from one highlighter into about 100 mL of water.

To make the drops I started by punching a hole in a food container with a needle, but wasn't able to achieve the consistency I wanted.
The best solution I found was to punch a hole in the bottom of the container and use a [fish tank aeration valve](http://www.amazon.com/Uxcell-Plastic-Aquarium-Control-Valves/dp/B00ZRHB5NW), sealing with super glue.
This was able to produce consistent drops, with an easily adjustable drop rate.

## Action
 
<iframe width="560" height="315" src="https://www.youtube.com/embed/y3eXu-68REc" frameborder="0" allowfullscreen></iframe>

It's a cool effect to see in person, but difficult to capture on camera because of the lighting conditions. Here are a few videos that do a better job of showing off the effect:  
[video 1](https://www.youtube.com/watch?v=rvY7NGncCgU)  
[video 2](https://www.youtube.com/watch?v=XwYIE-82i6Y)  
[video 3](https://www.youtube.com/watch?v=CaDtZA78uP0)  
