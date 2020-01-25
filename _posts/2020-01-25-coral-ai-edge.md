---
layout: post
title:  "Coral AI Edge TPUs"
sub_title: "First steps with the Coral USB Accelerator"
excerpt_separator:  <!-- more -->
categories:
  - Tinker
tags:
  - AI
  - Machine Learning
  - Google
  - Edge
---

After reading up some [nice stats](https://blog.raccoons.be/coral-tpu-jetson-nano-performance) on Google **Coral Edge TPUs** by Sam Sterckval at @RacoonsGroup, I got curious. Let's see how this works.

<!-- more -->

Here you can see his data on how Coral leaves NVIDIA's GTX1080 in the dust:

![](https://rscircus.github.io/assets/img/20200125_CoralBeatsGTX1080.png)

while graphing the fps these setups can do using the [MobileNetV2](https://arxiv.org/abs/1801.04381) classifier pre-trained on the ImageNet dataset. Yup, this is all about inference. And at this the Coral Edge TPU (let's call it CET) is really really good.

In this article we look at how to do inference with the Edge TPU, look at some perf characteristics and further explore, how the feel from a hobbyist perspective is.

> I hear and I forget. I see and I remember. I do and I understand.

_Disclaimer: This is a work in progress!_


## Hardware

My CET in shape of the USB Accelerator looks like this:

![](https://rscircus.github.io/assets/img/20200125_CoralPicture.jpg)

from the interesting side. And boring from the other, but you can look this up on the internets. Everything is soldered nothing has to be screwed together. The easiest step: just plug it in.

Let's `lsusb` it:

```
0> lsusb
[...]
Bus 003 Device 003: ID 1a6e:089a Global Unichip Corp.
[...]
```

Global Unichip Corp? That's [interesting](https://en.wikipedia.org/wiki/Global_Unichip_Corporation). GUC seems to be a knwo ASIC/SoC design foundry based in Taiwan. OK.


## Software

First, we need a rough plan:

- Setup the GUC hardware piece
- Install TensorFlow Lite
- Try to run a sample from one of the tutorials

### Setup

Beginning with https://www.coral.ai/docs/accelerator/get-started/ we need to install a binary blob (on Debian):

```
echo "deb https://packages.cloud.google.com/apt coral-edgetpu-stable main" | sudo tee /etc/apt/sources.list.d/coral-edgetpu.list
curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
sudo apt-get update
sudo apt-get install libedgetpu1-std
```

and trust Google that everything is OK with that one. 

It is recommended ot use a _USB 3.0 port_ for best perf and further there is another runtime which runs at 2x the clock frequency of `libedgetpu1-std` and can be installed like so:

```
sudo apt-get install libedgetpu1-max
```

### TensorFlow Lite

Now we install TensorFlow's API. I created a repo for this here [https://github.com/rscircus/coral.ai](https://github.com/rscircus/coral.ai).

### Example


## Sources

