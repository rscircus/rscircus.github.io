---
layout: post
title:  "k3sup on HypriotOS / RaspberryPi 3B+"
sub_title: "Setting up a kubernetes cluster at home"
categories:
  - Tinker
tags:
  - kubernetes
  - k3sup
  - k3s
  - RaspberryPi
---

## Motivation

> I hear and I forget. I see and I remember. I do and I understand.

Disclaimer: This is a work in progress!


## Hardware

The hardware is 4 RaspberryPis 3B+ assembled into a sweet little rack.

Power comes over PoE which the router can provide. Chose whatever you like. I've chosen the following setup:

| Type                                                    | Price/Piece |
| ------------------------------------------------------- | ----------- |
| TP-LINK SF1005P 5 Port 10/100 Desktop Switch with 4 PoE | ~40 E       |
| RaspberryPi 3B+                                         | ~40 E       |
| Sandisk Ultra 32Gb U1                                   | ~5 E        |
| DSLRKIT Active Splitter PoE 5V 2.4 A Micro USB Plug     | ~10 E       |
| Jun Raspberry Pi 3B+ Case Pack                          | ~25 E       |

and it looks like this:

![](https://rscircus.github.io/assets/img/MVIMG_20200121_122335.jpg)


## Software

As HypriotOS has docker already included, I was rooting for HypriotOs at the time of creation.

There is one pretty nice tutorial by them https://blog.hypriot.com/post/setup-kubernetes-raspberry-pi-cluster/ which assumes you are already connected to the tiny black-pearl.
Following that tutorial one sets up kubernetes in all its glory and resource hunger. It sucks. Don't do this. The note in my research journal from the time I did this reads: "Kubernetes seems to be rather hungry. Maybe I'll add a new masternode in the near future."

We start from first principles today.


### Preparing the Pis in a headless way.


#### Master node

We can start with one Pi. This approach makes the learning experience a little less frustrating. Using Hypriot's flash tool and cloud-init we flash the most recent image (as of date of writing):

``` 
flash -u wifi-user-data.yml --bootconf no-uart-config.txt hypriotos-rpi-v1.9.0.img.zip
```

which already takes care of plenty of things. However, you need to explore on Hypriot's page how to set up all the files going into the flash tool. See [here for more details](https://blog.hypriot.com/post/cloud-init-cloud-on-hypriot-x64/). The investment is worth it, as you can use these templates for the slave Pis later on. Further you can also use a more recent HypriotOs image.

Assuming our wifi config is OK we should be able to ping `black-pearl.yourdomain` in our network. To get an overview of our topology we can use `nmap` , e.g., 

``` 
sudo nmap -sn 192.168.1.0/24
```

This should give us the correct ip/name.

After selecting one of the nodes as master node, we log into it 

The default login data are:

```
user: pirate
pass: hypriot
```

clean up:

``` 
sudo apt remove --purge cloud-init
sudo apt autoremove
```

and update the OS:

``` 
sudo apt-get update
sudo apt-get upgrade
```

Due to many reasons I selected the master-node as the only node to have internet access and function as a router for the rest.

``` 
sudo apt-get install isc-dhcp-server
```

We can't go into all the details of setting up a DHCP. If you want a quick win, adapt [https://github.com/kubedge/kube-rpi/blob/master/config/cluster1/hypriotos/kubemaster-pi/etc/dhcp/dhcpd.conf](https://github.com/kubedge/kube-rpi/blob/master/config/cluster1/hypriotos/kubemaster-pi/etc/dhcp/dhcpd.conf).

Same goes for the ipforwarding setup. Adapt [https://github.com/kubedge/kube-rpi/blob/master/config/cluster1/hypriotos/kubemaster-pi/etc/sysctl.conf](https://github.com/kubedge/kube-rpi/blob/master/config/cluster1/hypriotos/kubemaster-pi/etc/sysctl.conf) for a quick win.

Keep in mind that the master node has to work as router from eth0 to wlan0 for the other nodes. For the NAT to work we need to configure it:

``` 
sudo sh -c "echo 1 > /proc/sys/net/ipv4/ip_forward"
sudo iptables -t nat -A POSTROUTING -o wlan0 -j MASQUERADE
sudo iptables -A FORWARD -i wlan0 -o eth0 -m state --state RELATED,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -i eth0 -o wlan0 -j ACCEPT
sudo sh -c "iptables-save > /etc/iptables.ipv4.nat"
```

After that, uncomment the iptable in `/etc/network/interfaces.d/eth0` .

To check if the routing is working we can plug our laptop into the (hardware) router and check if the master node assigns us an IP in the range `192.168.2.0/255` .

To check the setup (on linux) we can use nmcli on our laptop, e.g.:

``` 
nmcli dev show eth0
```

This will show us `routes` , `dns` , `domain` and `gateway` infos. Besides other things.


#### Slave nodes

Having our flash tool and config files prepared in the step above, things will get easier. I promise.

Next we flash our next sd-card without WLAN. After putting it back into the slave Pi we connect the slave Pi to the router with the cable and log into the master pi from our laptop. The master pi should have assigned an IP to the new slave Pi and we should be able to find it now (with nmap).

Jumphosting from the masternode into the slave now we clean up the installation again:

``` 
sudo apt-get remove --purge cloud-init
sudo apt-get autoremove
```

and update the OS:

``` 
sudo apt-get update
sudo apt-get upgrade
```

Following that we assign a new hostname:

``` 
sudo hostnamectl set-hostname blackbeard
```

I've chosen the characters from [Pirates of the Caribean](https://en.wikipedia.org/wiki/List_of_Pirates_of_the_Caribbean_characters). Here a selection: 'beckett', 'dalma', 'norrington'.

And now we update the dhcp conf file on the master node to assign a static IP to the slave node such that every time the cluster reboots we have the same network topology.
Considering that we might have to move around in the cluster this will ease the hassle. Actually I even noted the IP on the sd-card of each and every node with a permanent pen.

Keeping in mind that we have to restart the dhcp server every time after modifications:

``` 
sudo service isc-dhcp-server restart
```

## Conclusion

We set up a cluster using HypriotOs and configured DHCP and NAT. Further we have a static topology now with an up to date state of everything. In the next update of this blog post we use Alex Ellis' [k3sup](https://github.com/alexellis/k3sup) to get Rancher's minimal [k3s](https://k3s.io/) working by following with an [OpenFaas](https://www.openfaas.com/) setup.

## Sources

* https://blog.hypriot.com/
* https://kubedge.cloud/
* https://blog.alexellis.io/

