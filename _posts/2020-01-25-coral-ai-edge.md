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

> I hear and I forget. I see and I remember. I do and I understand.

_Disclaimer: This is a work in progress!_

Here you can see his data on how Coral leaves NVIDIA's GTX1080 in the dust:

![](https://rscircus.github.io/assets/img/20200125_CoralBeatsGTX1080.png)

while graphing the fps these setups can to using the [MobileNetV2](https://arxiv.org/abs/1801.04381) classifier pre-trained on the ImageNet dataset. Yup, this is all about inference. And at this, the Coral is really really good.

In this article we look at how to do inference with the Edge TPU, look at some perf characteristics and further explore, how the feel from a hobbyist perspective is.


## Hardware

It looks like this:

![](https://rscircus.github.io/assets/img/20200125_CoralPicture.jpg)

from the interesting side. And boring from the other, but you can look this up on the internets.


## Software

We need TensorFlow Lite to get started.


## Sources

