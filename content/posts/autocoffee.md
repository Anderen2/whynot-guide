+++
title = "Automated coffee ☕"
date = "2021-08-01T16:25:47+02:00"
author = "Andreas Skoglund"
authorTwitter = "" #do not include @
cover = "/hass-coffee-test.png"
tags = ["coffee", "homeassistant", "particle.io", "IOT", "microcontroller", "electronics", "particlecore", "Projects"]
keywords = ["key", "word"]
description = "Hacking an old capsule coffee-machine for better wakeups"
showFullContent = false
+++
<!-- <img style="text-align: center;">Home Assistant ☕</p> -->

The quest for coffee
===================
<img src="/dolce-gusto-melody.png" align="right" style="margin-left: 20px;"></img> 
Have you ever wanted to wake up with a automatic fresh cup of coffee (or tea) right from the bed every day? 

Do you have a capsule-style coffee-maker that now lives on the attic after you figured out the price of the capsules? 

Just enjoy reading random articles about coffee-machine hacks?

You find your current coffee maker to be just too convenient, and want your morning routine to be a bit more error prone? 

Then you have come to the right place. 
\
<br/>
## Demo
*Because no-one have ever seen a coffee machine in action before*
<iframe id="odysee-iframe" width="560" height="315" src="https://odysee.com/$/embed/coffeeclip/0e3d8075cd1dcb0ca859a29561dc811ae80bad11?r=HG1izTXpfieYdb738e7SpzG2wcGD153S" allowfullscreen></iframe>

*Youtube mirror in case Lbry/Odysee fails you: https://youtu.be/OcXusmTas7Y*

<!-- {{< figure src="/dolce-gusto-melody.png" >}} test -->
## How does it work? 
<a href="/dolce-gusto-melody-inside.png" target="_blank">
    <img src="/dolce-gusto-melody-inside.png" align="right" width=220px></img> 
</a>
With a mess of wires, some code hammered into a microcontroller with fingers crossed, and a dash of Home Assistant configuration is how. 
<br/><br/>

*image on the right is clickable for zooming, I wouldn't recommend it though*
<br/>

The microcontroller inside runs a state-machine-like code which allows it to move between the states illustrated below. In essence the user must replace the capsule and move the lever to the red side before it allows brewing. This allows some level of safety, as the machine refuses to run unless it has been manually prepared again after the previous brew. 

<a href="/coffee-states.svg" target="_blank">
    <img src="/coffee-states.svg" align="center" width=200px></img> 
</a>

*SVG image, click to zoom. Solid circles are internal states, dashed are external actions.*

<!-- {{< figure src="/coffee-states.svg" >}}
*SVG image, feel free to zoom* -->

## How could I do this too?

The first thing we do, is to toss out all the electronic circuitry inside except the pump, heater and the levers at the top. 

### The parts

#### Relay
The pump and heater are running on AC/mains input, so we need a relay to switch them. I used a 4 channel relay module here, as that was what I had laying around, but we need no more than 2 channels, this one would work: [2ch 5v relay module](https://www.banggood.com/Geekcreit-5V-1-or-2-or-4-or-8-or-16-Channel-Relay-Module-Optocoupler-For-PIC-AVR-DSP-ARM-DSP-p-1634909.html?warehouse=CN&ID=528396&p=0Q100667345353202112&custlinkid=1719718). 

#### Microcontroller
The microcontroller I used here is a [Particle.io Core](https://docs.particle.io/datasheets/discontinued/core-datasheet/), but any WiFi microcontroller could be used with some modifications on the code. I only used this as I had bought a couple of them some years ago. 

If you are able to read and change the code below yourself then I would recommend some sort of ESP-based controller like: [ESP8266](https://www.banggood.com/Geekcreit-Wireless-NodeMcu-Lua-CH340G-V3-Based-ESP8266-WIFI-Internet-of-Things-IOT-Development-Module-p-1420166.html?warehouse=CN&ID=522225&p=0Q100667345353202112&custlinkid=1719721) and run https://esphome.io. This would allow you to run without any cloud-integration too, which is a big advantage *(as waking up with the internet broken is when you need the coffee the most)*.

However, if you want to copy-paste the code below, or for some reason need your coffee machine to be available through a cloud, then the [Photon kit](https://store.particle.io/collections/wifi/products/photon) from the Particle-guys should be compatible. 

#### Powersupply
I used a dodgy 5v supply where the power-connections are way more close to each other than I am comfortable with, hopefully it goes up in flames soon such that I may replace it. I won't recommend it. 

I've seen others use the [HLK-PM01](https://www.banggood.com/HLK-PM01-AC-DC-220V-To-5V-Mini-Power-Supply-Module-Intelligent-Household-Switch-Power-Supply-Module-p-1242358.html?warehouse=CN&ID=0&p=0Q100667345353202112&custlinkid=1719722) before, so get that instead. 
*note, I've not tried this supply myself, do not blame me if your coffee (and machine) suddenly gets a bit too hot one morning*

#### Fuses
There is a thermal fuse installed on top of the heater already, and we can reuse this. If you accidentally blow it (as I did), then it may be replaced by a [SF129E](https://www.aliexpress.com/item/2043782114.html). Additionally I added a standard fuse for when the powersupply eventually gives up, here I used a fast-blowing 5a fuse (F5AL 250v). With it I had a cheap glass-fuse holder, similar to this ["Glass Tube Fuse Holder"](https://www.aliexpress.com/item/33054127144.html), but I found that this had melted when checking some time later. I guess I recommend using something else. 

#### Bonus: Refillable capsules
To avoid paying [Nestlé](https://en.wikipedia.org/wiki/Nestl%C3%A9#Controversy_and_criticisms) more than necessary for waking up in the morning, I highly recommend getting some [refillable capsules](https://www.banggood.com/3PcsSet-Colorful-Refillable-Coffee-Capsule-Cup-Reusable-Coffee-Pods-w-Spoon-Brush-for-Nescafe-Dolce-Gusto-Brewer-p-1603015.html?warehouse=CN&ID=47520&p=0Q100667345353202112&custlinkid=1719791). These may be filled with whatever you'd like, I've had success with instant coffee, filter-coffee grounds and tea. 

### My first attempt on a electronic schematic
{{< figure src="/autocoffee-circuit.svg" >}}
*SVG image, feel free to zoom*

### The code
Following is the code I wrote for the Particle Core microcontroller.
{{< gist anderen2 3201bbd45e67460050bab4463f5b6907 "coffee.ino" >}}
<!-- ```c
#include "math.h"

String stateStr;
``` -->
<!-- {{< figure src="/hass-coffee.png" >}} -->

<!-- {{< figure src="/coffee-states.svg" >}}
*SVG image, feel free to zoom* -->

<a href="/hass-coffee-test.png" target="_blank">
    <img src="/hass-coffee-test.png" align="right" width=220px></img> 
</a>

## Triggering our coffee
Okay, so now we have all these parts and code. How do we trigger it to do anything? 

If you are using a Particle.io controller as I did, then you first need to retrieve an [access token](https://docs.particle.io/tutorials/device-cloud/authentication/#access-tokens). 

Combining that, and whatever name/ID you've given your controller, we can use curl to trigger our machine as follows. With the last command, you should get 10s worth of coffee, which is +/- an espresso worth. Increasing the runtime will of course increase the amount of coffee you get. 

<br/>

```sh
# To get the current status of the coffee-maker, replace [DEVICE ID] and [ACCESS TOKEN]. 
curl -X GET https://api.particle.io/v1/devices/[DEVICE ID]/status?access_token=[ACCESS TOKEN]

# Brew 10 seconds worth of coffee, replace [DEVICE ID] and [ACCESS TOKEN]. 
curl -X POST -H "Accept: application/json, text/html" -H "Content-Type: application/x-www-form-urlencoded" https://api.particle.io/v1/devices/[DEVICE ID]/brew -d access_token=[ACCESS TOKEN] -d arg=10
```



### Home Assistant
If we want a more user friendly approach to creating our coffee (*and taking the first step into automating it*), we can use Home Assistant. 

The following configuration block goes into your configuration.yaml file and will provide you with two buttons for brewing coffee, one for a small coffee (20s) and one for a large (35s). You will also get a new sensor showing the current state of the machine. Remember to change all occurrences of `[DEVICE ID]` and `[ACCESS TOKEN]`. 

#### configuration.yml
{{< gist anderen2 2265dc3f167518562496cc4ad4d9d17f "homeassistant-particleio.yaml" >}}

## Congratulations! 
.. you now have yourself a overcomplicated coffee maker, that considering [Murphy](https://en.wikipedia.org/wiki/Murphy%27s_law), will likely break the moment you need it the most ;)

## Automating it
*Then for the fun part ...*

We now have all the basic blocks we need for automating this within Home Assistant, and you can easily do this with some [basic automation rules](https://www.home-assistant.io/getting-started/automation/). I'll let this one be a exercise for the reader, as figuring out when you want, and how to trigger your coffee is at least half the fun! 

For inspiration, I've hooked this together with Tasker on Android such that it automatically schedules and triggers 1 minute after my alarm goes off. If you want to know how, I'll explain it in details in the next post, this one is already long enough :) 

## Cheerio!
Thank you for sticking around, hope you found some form of use, or entertainment from this! Keeping with the theme, if you want to keep my capsules filled (or if you pity me my capsules and want me to get a real coffee): 

<a href='https://ko-fi.com/H2H27BCSP' target='_blank'><img height='36' style='border:0px;height:36px;' src='https://cdn.ko-fi.com/cdn/kofi2.png?v=3' border='0' alt='Buy Me a Coffee at ko-fi.com' /></a>