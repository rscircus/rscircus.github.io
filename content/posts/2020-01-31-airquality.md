---
categories:
- Tinker
date: "2020-01-31T00:00:00Z"
excerpt_separator: <!-- more -->
sub_title: Save your mind
tags:
- airquality
- sensors
- diy
- heatlh
- iot
title: Measuring air quality
---

Air quality has a huge impact on us humans. Our blood is in direct contact with the air through our lungs.

In this article we investigate the [nova PM sensor of type SDS011](http://aqicn.org/sensor/sds011/) from inovafit, which measures PM2.5 and PM10 particles.

<!--more-->


## Motivation

As the lung connects our surrounding air with our blood it is easy to transport all kinds of things in a quick way into the body and into the brain. Smokers enjoy the immediate drug delivery, for instance.
Recently it has been shown that the air we breath [gets dirtier](https://www.nytimes.com/interactive/2019/10/24/climate/air-pollution-increase.html) and the impacts on [learning humans](https://www.edworkingpapers.com/ai20-188) are investigated. By reasoning or extrapolating by having a deep look at the [Air Quality Index](https://en.wikipedia.org/wiki/Air_quality_index) it is clear that it is not healthy to breath dirty air. And dirty air is even [linked with depression](https://www.theguardian.com/environment/2019/dec/18/depression-and-suicide-linked-to-air-pollution-in-new-global-study) and [Alzheimers](https://www.theguardian.com/environment/2016/sep/05/toxic-air-pollution-particles-found-in-human-brains-links-alzheimers).

![](https://rscircus.github.io/assets/img/20200131_AirQualityLevels.png)

_via: http://aqicn.org/sensor/sds011/_

### A little bit of Medical Background

Even though PM is simply a measure of particle size, which in itself is not that dramatic, it can be used to identify particles which can enter the body via the human lung. The line of thought is that the critical size of < PM10 are the particles which can enter our blood because the lung can not defend itself anymore. And in our industrialized world most particles in that range are man made and not healthy.

## Hardware

What we have is a nova PM sensor SDS011 which comes with a USB dongle and looks like this:

![](https://rscircus.github.io/assets/img/20200131_AirQualitySensor.jpeg)

it also comes with [this datasheet](http://www.inovafitness.com/software/SDS011%20laser%20PM2.5%20sensor%20specification-V1.3.pdf).

## Software

Assuming you have a *nix machine, it's rather easy to read it out.

Basically this is all you need:

```python
# Be sure you have installed pyserial!
import serial

# Change this if the sensor lies somewhere else, but usually this works.
# you can search for it via "udevadm info -q property --export /dev/ttyUSB0"
# by trying various ttys
dev = serial.Serial('/dev/ttyUSB0', 9600)

if not dev.isOpen():
    dev.open()

msg = dev.read(10)
# Check for correct byte order
assert msg[0] == ord(b'\xaa')
assert msg[1] == ord(b'\xc0')
assert msg[9] == ord(b'\xab')

# The interesting values lie here
pm25 = (msg[3] * 256 + msg[2]) / 10.0
pm10 = (msg[5] * 256 + msg[4]) / 10.0

# Validate our reading
checksum = sum(v for v in msg[2:8]) % 256
assert checksum == msg[8]

print(f"'PM10': {pm10}, 'PM2_5': {pm25}")
```

After this you can do various things. For instance: Wrap the above code in a function, `watch` it and create diagrams like this:

![](https://rscircus.github.io/assets/img/20200131_AirQualityReading.jpeg)

where you can see the reaction of the sensor to a simple match being lit in a tiny room ca. 30cm away from the sensor.

We can use this [gist](https://gist.github.com/marw/9bdd78b430c8ece8662ec403e04c75fe) and run the program with

```bash
watch python ./read_nova_pm_sensor.py --csv match_experiment.csv
```

to use the 'recorded' `csv` to generate the plot above using LibreOffice Calc.

## Conclusion

We read out the SDS011 and collected a set of timestamps. After lighting a match we validated its functionality. In the next part we wrap a little more code around this tiny start and let it plot data automagically for us.
