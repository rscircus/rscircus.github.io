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

_Disclaimer: This is a work in progress!_


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
flash -u wifi-user-data.yml --bootconf no-uart-config.txt HypriotOS-rpi-v1.9.0.img.zip
```

which already takes care of plenty of things. However, you need to explore on Hypriot's page how to set up all the files going into the flash tool. See [here for more details](https://blog.hypriot.com/post/cloud-init-cloud-on-hypriot-x64/). The investment is worth it, as you can use these templates for the slave Pis later on. Further you can also use a more recent HypriotOS image.

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

We can't go into all the details of setting up a DHCP. If you want a quick win, adapt [https://github.com/kubedge/kube-rpi/blob/master/config/cluster1/HypriotOS/kubemaster-pi/etc/dhcp/dhcpd.conf](https://github.com/kubedge/kube-rpi/blob/master/config/cluster1/hypriotos/kubemaster-pi/etc/dhcp/dhcpd.conf).

Same goes for the ipforwarding setup. Adapt [https://github.com/kubedge/kube-rpi/blob/master/config/cluster1/HypriotOS/kubemaster-pi/etc/sysctl.conf](https://github.com/kubedge/kube-rpi/blob/master/config/cluster1/hypriotos/kubemaster-pi/etc/sysctl.conf) for a quick win.

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

Further it makes sense to copy your ssh keys through the whole cluster using

```
ssh-copy-id pirate@192.168.2.1 # and so on...
```

It will come in handy later with k3sup and while maintaining the thing.

## Intermission (previous Conclusion)

We set up a cluster using HypriotOS and configured DHCP and NAT. Further we have a static topology now with an up to date state of everything. In the next update of this blog post we use Alex Ellis' [k3sup](https://github.com/alexellis/k3sup) to get Rancher's minimal [k3s](https://k3s.io/) working by following with an [OpenFaas](https://www.openfaas.com/) setup. Potentially we also intro how to administer this cluster using [Ansible](https://www.ansible.com/).

## k3sup

The following chapter will be a bit more detailed as this is not standard admin stuff. Even though I know people who claim this is the case, but then there is [memenetes](https://twitter.com/memenetes). To me it still feels pretty new. 🤷‍♀️

[k3sup](https://github.com/alexellis/k3sup) makes bootstrapping kubernetes very easy. It is a static Go binary. Hence, installing is basically copying it.

First we install k3sup on our laptop which is connected to the master Pi.

```
curl -sLS https://get.k3sup.dev | sh
sudo install k3sup /usr/local/bin/
```

Our master Pi from above will be the Kubernetes master and the nodes will be the agents to our Kubernetes.

First the master:

```
k3sup install --ip 192.168.2.1 --user pirate
```

which does some automagic installation for you. Luckily we get informed about what is going on:

```
Running: k3sup install
Public IP: 192.168.2.1
ssh -i /home/rawland/.ssh/id_rsa -p 22 pirate@192.168.2.1
ssh: curl -sLS https://get.k3s.io | INSTALL_K3S_EXEC='server  --tls-san 192.168.2.1 ' INSTALL_K3S_VERSION='v1.0.1' sh -

[INFO]  Using v1.0.1 as release
[INFO]  Downloading hash https://github.com/rancher/k3s/releases/download/v1.0.1/sha256sum-arm.txt
[INFO]  Downloading binary https://github.com/rancher/k3s/releases/download/v1.0.1/k3s-armhf
[INFO]  Verifying binary download
[INFO]  Installing k3s to /usr/local/bin/k3s
[INFO]  Creating /usr/local/bin/kubectl symlink to k3s
[INFO]  Skipping /usr/local/bin/crictl symlink to k3s, command exists in PATH at /usr/bin/crictl
[INFO]  Creating /usr/local/bin/ctr symlink to k3s
[INFO]  Creating killall script /usr/local/bin/k3s-killall.sh
[INFO]  Creating uninstall script /usr/local/bin/k3s-uninstall.sh
[INFO]  env: Creating environment file /etc/systemd/system/k3s.service.env
[INFO]  systemd: Creating service file /etc/systemd/system/k3s.service
[INFO]  systemd: Enabling k3s unit
Created symlink /etc/systemd/system/multi-user.target.wants/k3s.service → /etc/systemd/system/k3s.service.
[INFO]  systemd: Starting k3s
Result: [INFO]  Using v1.0.1 as release
[INFO]  Downloading hash https://github.com/rancher/k3s/releases/download/v1.0.1/sha256sum-arm.txt
[INFO]  Downloading binary https://github.com/rancher/k3s/releases/download/v1.0.1/k3s-armhf
[INFO]  Verifying binary download
[INFO]  Installing k3s to /usr/local/bin/k3s
[INFO]  Creating /usr/local/bin/kubectl symlink to k3s
[INFO]  Skipping /usr/local/bin/crictl symlink to k3s, command exists in PATH at /usr/bin/crictl
[INFO]  Creating /usr/local/bin/ctr symlink to k3s
[INFO]  Creating killall script /usr/local/bin/k3s-killall.sh
[INFO]  Creating uninstall script /usr/local/bin/k3s-uninstall.sh
[INFO]  env: Creating environment file /etc/systemd/system/k3s.service.env
[INFO]  systemd: Creating service file /etc/systemd/system/k3s.service
[INFO]  systemd: Enabling k3s unit
[INFO]  systemd: Starting k3s
 Created symlink /etc/systemd/system/multi-user.target.wants/k3s.service → /etc/systemd/system/k3s.service.

ssh: sudo cat /etc/rancher/k3s/k3s.yaml
[...]
```

In the end the cat prints out how `k3s` is configured on the master Pi. We can also have a look at it by logging into the master Pi and

```
sudo kubectl get node -o wide
```
or executing it locally on our laptop, if we have `kubectl` installed. Sans `sudo` in the latter case.

This should show us some Docker-ish summary of k8s. To summarize what just happened: `k3sup` installed Rancher's k3s + a bunch of scripts on the master Pi. If you mess things up, going back to normal is basically `sudo k3-uninstall.sh`. As we witnessed, `k3sup` somehow determined what variant of `k3s` had to be installed with the given hardware (`armhf`), which is a nice thing. Further we received a kubeconfig which was also stored locally. I trimmed this away above. The kubeconfig also sits in `/etc/rancher/k3s/k3s.yaml` on the master Pi.

Now, back on our laptop, we join the other agents/nodes to our cluster:

```
k3sup join --ip 192.168.2.101 --server-ip 192.168.2.1 --user pirate
```

and this for every node. Which is `192.168.2.101-103` for me. If you want to automate this, you can also create a simple BASH script. It can happen that the latter command finishes with the following error:

```
Error: unable to connect to 192.168.2.101:22 over ssh: ssh: handshake failed: ssh: unable to authenticate, attempted methods [none publickey], no supported methods remain
```

which means that we didn't copy our ssh key over with `ssh-copy-id` (see above). If everything went fine we see four entries in the return from `sudo kubectl get node -o wide`.

At this point we can deploy stuff on Kubernetes if we want to. 😊

### Kubernetes Dashboard

To get an overview of the state of our kubernetes cluster, we install the dashboard.

For this we need `kubectl` on our laptop. To install it, when you are on something Debian-ish:

```
sudo apt-get update && sudo apt-get install -y apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubectl
```

```
k3sup app install kubernetes-dashboard
```

which results in

```
0> k3sup app install kubernetes-dashboard
Using kubeconfig: /home/rawland/kubeconfig
armNode architecture: "arm"
namespace/kubernetes-dashboard unchanged
serviceaccount/kubernetes-dashboard unchanged
service/kubernetes-dashboard unchanged
secret/kubernetes-dashboard-certs unchanged
secret/kubernetes-dashboard-csrf configured
Warning: kubectl apply should be used on resource created by either kubectl create --save-config or kubectl apply
secret/kubernetes-dashboard-key-holder configured
configmap/kubernetes-dashboard-settings unchanged
role.rbac.authorization.k8s.io/kubernetes-dashboard unchanged
clusterrole.rbac.authorization.k8s.io/kubernetes-dashboard unchanged
rolebinding.rbac.authorization.k8s.io/kubernetes-dashboard unchanged
clusterrolebinding.rbac.authorization.k8s.io/kubernetes-dashboard unchanged
deployment.apps/kubernetes-dashboard unchanged
service/dashboard-metrics-scraper unchanged
deployment.apps/dashboard-metrics-scraper unchanged
error: must specify one of -f and -k
error: must specify one of -f and -k
=======================================================================
= Kubernetes Dashboard has been installed.                                        =
=======================================================================

#To forward the dashboard to your local machine
kubectl proxy

#To get your Token for logging in
kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep default-token | awk '{print $1}')

# Once Proxying you can navigate to the below
http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/#/login

Thanks for using k3sup!
```

for me.

At this point it looks like something is broken here. As there are two lines containing: `error: must specify one of -f and -k`, which means: Have a look at the code, what the heck is going on here. For now I simply ignore it and note down to maybe help out at the repo (TODO).

Trying the recommendations to `kubectl proxy` the dashboard to our localhost (laptop), however, results in a seemingly functioning Kubernetes Dashboard:

![](https://rscircus.github.io/assets/img/20200121_KubernetesDashboard.png)


To get the token we can:

```
sudo kubectl -n kubernetes-dashboard describe secret $(sudo kubectl -n kubernetes-dashboard get secret | grep default-token | awk '{print $1}')
```

on the master Pi, as the defaults following the manual on the laptop fail with:

```
0> kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep default-token | awk '{print $1}')
The connection to the server localhost:8080 was refused - did you specify the right host or port?
The connection to the server localhost:8080 was refused - did you specify the right host or port?
```

At this point it is pretty likely that `k3sup` is clashing with `HypriotOS` considering the `kubernetes-dashboard` installation.

![](https://rscircus.github.io/assets/img/20200121_KubernetesFail.png)


### Cleaning up

At this point in time, it's time to see if we can clean up. (TODO: Diff the whole image maybe?)

On the master Pi we proceed by:

```
k3s-killall.sh && k3s-uninstall.sh
```

And the agents/nodes:

```
k3s-killall.sh && k3s-agent-uninstall.sh
```

which looks rather robust.

## Conclusion

We used `k3sup` to set up a Kubernetes cluster which worked out nicely. Unfortunately we failed to install the `kubernetes-dashboard` using `k3sup` and have to debug what is going on there. Most likely a simple authorization problem. Further, it has to be mentioned that `k3sup` is a one-way route, as there is no direct way to uninstall things using `k3sup` as far as I see.

## Sources

* [https://blog.hypriot.com/](https://blog.hypriot.com)
* [https://kubedge.cloud](https://kubedge.cloud)
* [https://blog.alexellis.io](https://blog.alexellis.io)

