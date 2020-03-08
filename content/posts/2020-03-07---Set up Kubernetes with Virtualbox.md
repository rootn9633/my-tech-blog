---
title: Set up Kubernetes with Virtualbox
date: "2020-03-07T22:12:03.284Z"
template: "post"
draft: false
slug: "/posts/Set-up-Kubernetes-with-Virtualbox/"
category: "Kubernetes"
tags:
  - "Kubernetes"
  - "Virtualbox"
  - "Ubuntu"
description: "The steps for setting up Kubernetes with Virtualbox VMs"
---

Seeing that there are so much hype about Kubernetes, I decided to try it myself and see what it could do.

As I am trying to learn as much as I can about the mechanics behind it, I decided to create VMs in VirtualBox to have the experience of setting up Kubernetes by scratch.

I came upon [this guide](https://medium.com/@KevinHoffman/building-a-kubernetes-cluster-in-virtualbox-with-ubuntu-22cd338846dd) detailing the steps of doing so. Below are the documentation of my learning progress.

## Setup the VMs

### Create host only network

![image-20200307235659763](/home/rootn/.config/Typora/typora-user-images/image-20200307235659763.png)

### Create first VM (kubemaster)

I created a VM with

* system: Ubuntu server 18.04
* CPU: 1
* Memory: 1Gb

#### disable swap

It is recommended to disable the swap on the nodes, see reasons [here](https://serverfault.com/questions/881517/why-disable-swap-on-kubernetes)

1. list swap files with `cat /proc/swaps`
2. Turn off all swap devices and files with `swapoff -a`
3. Remove any matching reference found in `/etc/fstab`

[source](https://serverfault.com/questions/684771/best-way-to-disable-swap-in-linux)

#### Install docker

I followed the official documentation on how to install docker found [here](https://docs.docker.com/install/linux/docker-ce/ubuntu/)

### Clone 2 VMs (worker1, worker2)

make 2 clones of the kubemaster VM in VirtualBox

### Configure the VMs

For quick development purposes I configured the VMs so that they each have two networks:

* Nat: for connection to the host computer.
* host-only network: to communicate with each other

#### Configure port forwarding for the NAT

In VirtualBox, I set up the port forwarding for each VM so that when I ssh via three different ports, I can connect to each VM respectively.

#### Give each VM a static IP address in the host-only network

Edit the netplan configuration file and add the following under the `ethernets` section. Remember to change the `addresses` section to match your setting, and give each VM a unique IP.

```text
enp0s8:
    dhcp4: no
    addresses: [192.168.56.100/24]
```

Here I have the following IPs to each VM:

* kubemaster: 192.168.56.100
* worker1: 192.168.56.101
* worker2: 192.168.56.102

You can check your configuration by making sure that you can ssh into each vm respectively via the host only network.

#### Configure the hostname

Follow the below steps to change the host name of each machine

1. edit the `/etc/hostname` file
2. change each occurrence in the `/etc/hosts` file
3. reboot

[source](https://www.cyberciti.biz/faq/ubuntu-change-hostname-command/)

#### Result

Now I will have three machines configured as follows: (note that I changed the number of CPUs for **kubemaster** to 2)

| VM name    | hostname   | host-only network ip | CPUs | Memory |
| ---------- | ---------- | :------------------- | ---- | ------ |
| kubemaster | kubemaster | 192.168.56.100       | 2    | 2Gb    |
| worker1    | worker1    | 192.168.56.101       | 1    | 1Gb    |
| worker2    | worker2    | 192.168.56.102       | 1    | 1Gb    |

## Setup Kubernetes

### Setup kubemaster

#### Install kubeadm on kubemaster

I followed this [guide](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/) on installing `kubeadm`

#### Set up the control-plane

On the master, I ran the following to start up a cluster. Remember to change the `--apiserver-advertise-address` to match your settings, or else the pods won't be able to find the master.

```shell
kubeadm init --apiserver-advertise-address=192.168.56.100 --pod-network-cidr=192.168.0.0/16
```

Since I am going to use Calico, I need to set the pod network using CIDR.

The setting means that the **nodes** are getting the IPs 192.168.56.100-102, and the **Pods** will get the IPs 192.168.0.0/16

After the `kubeadm` finished, you should see something like the following.

```text
Your Kubernetes control-plane has initialized successfully!
```

It would also give you a command for `kubeadm join`, **you must remember this command**. Saving it is a good idea.

#### Setup Calico as the pod network

Install Calico with the following command

```shell
kubectl apply -f https://docs.projectcalico.org/v3.9/manifests/calico.yaml
```

Confirm that all of the pods are running with the following command.

```shell
watch kubectl get pods --all-namespaces
```

Wait until each pod has the `STATUS` of `Running`.

### Setup the workers

#### Install kubeadm on worker

As with **kubemaster**, I followed this [guide](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/) on installing `kubeadm`

#### Join the cluster

Remember the `kubeadm join` command you just saved? Run the command on the workers. Once finished, go back to the **kubemaster** and run `watch kubectl get pods --all-namespaces`. You should see your pods joining with the output something like below:

```text
NAMESPACE     NAME                                       READY   STATUS              RESTARTS
  AGE
kube-system   calico-kube-controllers-6b9d4c8765-5997r   1/1     Running             0
  21m
kube-system   calico-node-hs55k                          1/1     Running             0
  21m
kube-system   calico-node-v6zh5                          1/1     Running             0
  21m
kube-system   calico-node-x4jjr                          1/1     Running             0
  16m
kube-system   coredns-6955765f44-8lthn                   1/1     Running             0
  34m
kube-system   coredns-6955765f44-q67vz                   1/1     Running             0
  34m
kube-system   etcd-kubemaster                            1/1     Running             0
  34m
kube-system   kube-apiserver-kubemaster                  1/1     Running             0
  34m
kube-system   kube-controller-manager-kubemaster         1/1     Running             0
  34m
kube-system   kube-proxy-84g22                           1/1     Running             0
  34m
kube-system   kube-proxy-mkt42                           1/1     Running             0
  16m
kube-system   kube-proxy-nvjcj                           1/1     Running             0
  32m
kube-system   kube-scheduler-kubemaster                  1/1     Running             0
  34m
```
