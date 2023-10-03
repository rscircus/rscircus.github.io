---
categories:
- Tinker
date: "2020-02-29T00:00:00Z"
draft: true
excerpt_separator: <!-- more -->
sub_title: Recycle your old disks and get yourself a modular low-cost NAS
tags:
- nas
- SoC
title: Setting up a NAS using the Odroid HC1
---

The [Odroid-HC2: Home Cloud One](https://www.hardkernel.com/shop/odroid-hc1-home-cloud-one/) by hardkernel is an interesting piece of hardware. It sports an Arm architecture called *big.LITTLE* where a relatively battery-saving CPU (LITTLE) is combined with a more powerful CPU (big). The Odroid comes with a Samsung Exynos5422 ARM® Cortex™-A15 Quad 2.0GHz/Cortex™-A7 Quad 1.4GHz.

We will create an extensbile headless NAS with as little as effort as possible.

![](https://rscircus.github.io/assets/img/20200229_Odroid_NAS_stack.jpg)

<!--more-->

## Odroid HC1

The main features are

- Samsung Exynos 5422 eight-core CPU
- 2GB DDR3 RAM
- USB3.0-based SATA3 port
- Gigabit ethernet (Realtek chipset)
- USB2.0 port
- MicroSD slot
- UART for serial console
- Stackable aluminum frame that acts as a heatsink and drive cage
- No HDMI
- No WLAN Chip

We see now that it is missing a wifi chip. The idea is to plug the NAS directly into the router for some better data transmission perf. However, we will use a [very cheap and small USB Wifi Stick](https://smile.amazon.de/gp/product/B008IFXQFU/ref=ppx_yo_dt_b_asin_title_o03_s00?ie=UTF8&psc=1). It comes at "5.79 €" last time I checked. (2020.02.29)

## OS Image

We'll be using the OMV ready made image created by openmediavault. Which is available [here](https://sourceforge.net/projects/openmediavault/files/OMV%204.x%20for%20Single%20Board%20Computers/OMV_4_Odroid_XU4_HC1_HC2.img.xz/download)

After we download it, we unzip and flash it:

```bash
sudo unxz OMV_4_Odroid_XU4_HC1_HC2.img.xz
sudo dd status=progress bs=4M if=OMV_4_Odroid_XU4_HC1_HC2.img of=/dev/sdX
sudo sync
```

Following that your SD card will look like this:

![](https://rscircus.github.io/assets/img/20200229_Odroid_SDCard.png)

Following that flashing we boot it. As OMV has to set itself up, the first boot can take up to 30 minutes.

Start: 15:57

## Sources

- https://loganmarchione.com/2018/06/odroid-hc2-as-an-entry-level-nas/
