---
layout: post
title:  "Getting started with Coral Edge TPUs"
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

from the interesting side.

And boring from the other, but you can look this up on the internets. Everything is soldered, nothing has to be screwed together. Feels like a day of vacation. 🏝️ The easiest step: just plug it in.

Let's `lsusb` it:

```
0> lsusb
[...]
Bus 003 Device 003: ID 1a6e:089a Global Unichip Corp.
[...]
```

Global Unichip Corp? That's [interesting](https://en.wikipedia.org/wiki/Global_Unichip_Corporation). The Global Unichip Corporation seems to be a known ASIC/SoC design foundry based in Taiwan.


## Software

First, we need a rough plan:

- Setup the GUC hardware piece
- Install TensorFlow Lite
- Try to run a sample from one of the tutorials

### Setup

According to [https://www.coral.ai/docs/accelerator/get-started/](https://www.coral.ai/docs/accelerator/get-started) we need to install a binary blob (on Debian):

```
echo "deb https://packages.cloud.google.com/apt coral-edgetpu-stable main" | sudo tee /etc/apt/sources.list.d/coral-edgetpu.list
curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
sudo apt-get update
sudo apt-get install libedgetpu1-std
```

and trust Google that everything is OK with that one.

It is recommended to use an _USB 3.0 port_ for best perf and further there is another runtime which runs at 2x the clock frequency of `libedgetpu1-std` and can be installed like so:

```
sudo apt-get install libedgetpu1-max
```

During the installation of the `max`-version we are asked if we understand physics. We confirm with yes and move on. If you plugged the USB Accelerator into your machine ahead of installing the `libedge`. Plug it out and in again, else you might get a `ValueError: Failed to load delegate from libedgetpu.so.1` error during runtime later on.


### TensorFlow Lite

Now we install the TensorFlow Runtime. I created a repo for this here [https://github.com/rscircus/coral.ai](https://github.com/rscircus/coral.ai) and we'll be having a look at the [TensorFlow Lite Python quickstart](https://www.tensorflow.org/lite/guide/python) as a guide but set things up in a potentially less messy manner using [poetry](https://python-poetry.org/).

First we create it using poetry:

```
pyenv install 3.7.0  # Everything works with Python <=3.7.0
mkdir coral.ai && cd coral.ai
pyenv local 3.7.0    # Tells the shell to use 3.7.0 in this directory (and subdirs)
poetry init          # Follow the guide which starts with this command
poetry shell         # We shell into the virtualenv
```

Then we add the most recent version of `tflite_runtime`, which we can find [here](https://www.tensorflow.org/lite/guide/python):

For instance

```
wget https://dl.google.com/coral/python/tflite_runtime-1.14.0-cp37-cp37m-linux_x86_64.whl
```

Then we install it using

```
pip install https://dl.google.com/coral/python/tflite_runtime-1.14.0-cp37-cp37m-linux_x86_64.whl
```

OK, we have `tflite_runtime (1.14.0)` and everything contained in a nice virtualenv with `py3.7`.


### Example

Now we clone the example:

```
mkdir example && cd example
git clone https://github.com/google-coral/tflite.git
```

followed by `cd`-ing into the parrot example

```
cd example/tflite/python/examples/classification
```

and have a look at what this installs:

```
cat install_requirements.sh
```

![](https://rscircus.github.io/assets/img/20200125_CoralInstallReqs.png)

OK, this pulls various stuff and also modifies the Python installation we have. Let's delete the `pip` line and let poetry manage everything:

```
#!/bin/bash
#
# Copyright 2019 Google LLC
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     https://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
readonly SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
readonly TEST_DATA_URL=https://github.com/google-coral/edgetpu/raw/master/test_data

# Install required Python packages,
# but not on Mendel (Dev Board)—it has these already and shouldn't use pip
if [[ ! -f /etc/mendel_version ]]; then
  poetry add numpy Pillow
fi

# Get TF Lite model and labels
MODEL_DIR="${SCRIPT_DIR}/models"
mkdir -p "${MODEL_DIR}"

(cd "${MODEL_DIR}" && \
curl -OL "${TEST_DATA_URL}/mobilenet_v2_1.0_224_inat_bird_quant_edgetpu.tflite" \
     -OL "${TEST_DATA_URL}/mobilenet_v2_1.0_224_inat_bird_quant.tflite" \
     -OL "${TEST_DATA_URL}/inat_bird_labels.txt")

# Get example image
IMAGE_DIR="${SCRIPT_DIR}/images"
mkdir -p "${IMAGE_DIR}"

(cd "${IMAGE_DIR}" && \
curl -OL "${TEST_DATA_URL}/parrot.jpg")
```

with this modified `install_requirements.sh` we execute it:

```
./install_requirements.sh
```

Then we do our first parrot identification with

```
(coral.ai-KP9wyUS_-py3.7) 0> python classify_image.py --model models/mobilenet_v2_1.0_224_inat_bird_quant_edgetpu.tflite --labels models/inat_bird_labels.txt --input images/parrot.jpg
INFO: Initialized TensorFlow Lite runtime.
----INFERENCE TIME----
Note: The first inference on Edge TPU is slow because it includes loading the model into Edge TPU memory.
11.1ms
2.6ms
2.5ms
2.6ms
2.6ms
-------RESULTS--------
Ara macao (Scarlet Macaw): 0.76172
```

which is this beauty:

![](https://www.coral.ai/static/docs/images/parrot.jpg)

Ha! Great success! And the true fun begins. 😊

## Conclusion

We set up the CET and created a Python based virtualenv which hosts the `tflite_runtime`. Using this setup we classified the *ara macao* in a picture at `0.76172` confidence.

In the next step we want to create and train a model which will be deployed to the Coral hardware like so:

![](https://www.coral.ai/static/docs/images/edgetpu/compile-workflow.png)

and look at some perf characteristics.


## Sources

- https://www.coral.ai/docs
- https://blog.raccoons.be/coral-tpu-jetson-nano-performance
- https://www.tensorflow.org/lite

