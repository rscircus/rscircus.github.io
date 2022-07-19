---
layout: page
title:  "Open Source Hearing Aids"
sub_title: "New hardware and smartphones are changing the game..."
excerpt_separator:  <!-- more -->
tags:
  - Accessibility
  - Hardware and Software Love
---

State of Open Source Hearing Aids and SOTA in Hearing Aids.

<!-- more -->

## Overview

This writing is concerned with the SOTA of hearing aids and focuses on Open Source implementations of known algorithms in this area. 

## Disclaimer

Basically this document is a summary of my excerpts from personal notes created in 07.2022.

## MaixPy - A MicroPython implementation for the Kendryte K210 RISC-V ("Edge AI") architecture

A project created by SiPeed: https://maixpy.sipeed.com

From a developers perspective it is very easy to setup a noise map and configure an microphone array to achieve results like the following:

<video src="https://user-images.githubusercontent.com/1167114/179713272-5d92e689-66e1-475b-937c-da9339508206.mp4" controls="controls" style="max-width: 730px;">
</video>

Depicted is the localization using a Maix Go with Kendryte's K210. The K210 is meanwhile succeeded by the K510 - while Kendryte is now known as https://canaan.io.

The code needed is simply this:

```python
from Maix import MIC_ARRAY as mic
import lcd

lcd.init()
mic.init()#Default configuration
# mic.init(i2s_d0=23, i2s_d1=22, i2s_d2=21, i2s_d3=20, i2s_ws=19, i2s_sclk=18, sk9822_dat=24, sk9822_clk=25)#Customizable configuration IO

while True:
    imga = mic.get_map() # Get sound source distribution image
    b = mic.get_dir(imga) # Calculate and get the sound source direction
    a = mic.set_led(b,(0,0,255))# Configure RGB LED color value
    imgb = imga.resize(160,160)
    imgc = imgb.to_rainbow(1) # Convert image to rainbow image
    a = lcd.display(imgc)
mic.deinit()
```

When drilling down into the code we find a `C` implementation in: https://github.com/sipeed/MaixPy/blob/master/components/micropython/port/src/Maix/Maix_mic_array.c

There exist two interesting functions:

```C
STATIC mp_obj_t Maix_mic_array_get_map(void)
{
    image_t out;

    out.w = 16;
    out.h = 16;
    out.bpp = IMAGE_BPP_GRAYSCALE;
    out.data = xalloc(256);
    mic_done = 0;

    volatile uint8_t retry = 100;

    while(mic_done == 0)
    {
        retry--;
        msleep(1);
    }

    if(mic_done == 0 && retry == 0)
    {
        xfree(out.data);
        mp_raise_OSError(MP_ETIMEDOUT);
        return mp_const_false;
    }

    memcpy(out.data, thermal_map_data, 256);

    return py_image_from_struct(&out);
}
```

and

```C
STATIC mp_obj_t Maix_mic_array_get_dir(size_t n_args, const mp_obj_t *pos_args, mp_map_t *kw_args)
{
    uint8_t led_brightness[12]={0};

    image_t *arg_img = py_image_cobj(pos_args[0]);
    PY_ASSERT_TRUE_MSG(IM_IS_MUTABLE(arg_img), "Image format is not supported.");

    if(arg_img->w!=16 || arg_img->h!=16 || arg_img->bpp!=IMAGE_BPP_GRAYSCALE)
    {
        mp_raise_ValueError("image type error, only support 16*16 grayscale image");
        return mp_const_false;
    }

    calc_voice_strength(arg_img->data, led_brightness);

    mp_obj_t *tuple, *tmp;

    tmp = (mp_obj_t *)malloc(12 * sizeof(mp_obj_t));

    for (uint8_t index = 0; index < 12; index++)
        tmp[index] = mp_obj_new_int(led_brightness[index]);

    tuple = mp_obj_new_tuple(12, tmp);

    free(tmp);

    return tuple;
}
```

Further, it is interesting to note, that the `MIC_ARRAY` is generating the heatmap, called `thermal_map_data` representing a `16x16` bits image as 256 bit long array. What is also interesting is that the direction indication, i.e., the brightness of each LED on the ring is determined by using the heatmap in the `calc_voice_strength` function.

Impressive, how far we get with these simple methods. Most interesting is how the microphone array generates this heatmap, which is not obvious from the code itself.

## The Open Master Hearing Aid


One of the most interesting projects is coming from the Kompetenzzentrum fuer Hoergeraete-Systemtechnik. It's especially remarkable, as [Electrical Engineers are fading form the job market](https://www.theregister.com/2022/07/18/electrical_engineers_extinction/) because [Software is Eating all the jobs](https://techcrunch.com/2011/08/21/software-is-eating-all-the-jobs-too/?guccounter=1&guce_referrer=aHR0cHM6Ly9kdWNrZHVja2dvLmNvbS8&guce_referrer_sig=AQAAAL-YyfJ5cae1TydJyH63UBHtJBo4xnJoLf71Rtw-GFvx4uBP6co_nINvHL85VIePQ92TG59vPjcz74C1JLMzCcXWHKGdJuzj_jofDHdKgiPKlXgpn0gVTkLJm8EvDdEmrDG8YhWXQ7Aw2veBFUBK8XOM1xG_nfPRbzlc8JpCkXHL).

### Motivation

From the primer:

>The HörTech open Master Hearing Aid (openMHA), is a development and evaluation software platform that is able to execute hearing aid signal processing in real-time on standard computing hardware with a low delay between sound input and output.

=> Real-time and low latency are key! Else this will mess with the brain.

### Structure

- Runs in Matlab/Octave
- Uses JACK -- even though `*nix` is already far into [PulseAudio](https://www.freedesktop.org/wiki/Software/PulseAudio/Documentation/User/Community/) while [PipeWire](https://pipewire.org) / [PipeWire - Repo](https://gitlab.freedesktop.org/pipewire/pipewire) gets more and more traction... 🙈
- Architecture:
    ![](https://rscircus.github.io/assets/img/Pasted_image_20220719164329.png)
- Above we see four components:
	- **MHA** = CLI application which sets everything up
		- loads (openMHA) plugins
		- i/o modules
		- manual configuration interface
		- TCP/IP based configuration interface
	- **openMHA plugins** or simply plugins = provide the DSP or ASP capabilities.
		- Usually: `1 algorithm <-> 1 plugin`
	- **I/O** modules
	- **libopenmha** - dynamic library to connect to kernel
- _Summary:_ Virtual hearing aid processing can be achieved by combination of many plugins


### How it works

- The audio signal is processed in chunks of defined length called _fragments_. These can be processed/transported either as:
	- audio samples in time domain
	- spectra of fragments in the STFT domain
	- these are passed through the plugins
- Recommendation:
	- Sharing STFT data lowers latency!
	- Sharing of non-audio information is also possible (via so-caled _AC variables_)

### Why this project is awesome

- They provide SD card images for the Beaglebone Black w/ the Cape4all soundcard.
-  They provide a basic hearing aid processing chain covering
	-   bilateral adaptive differential microphones for noise suppression
	-   binaural coherence filter for feedback reduction and dereverberation
	-   multi-band dynamic range compressor for hearing loss compensation
	-   spatial filtering algorithms:
	    -   a delay-and-sum beamformer
	    -   a MVDR beamformer
	-   single-channel noise reduction
	-   resampling and filter plugins
	-   STFT cyclic aliasing prevention
	-   adaptive feedback cancellation
	-   probabilistic sound source localization


### Getting started

The starting guide linked below has a few example `*.cfg` files for some simple applications.

### Sources

- http://www.openmha.org/docs/openMHA_starting_guide.pdf