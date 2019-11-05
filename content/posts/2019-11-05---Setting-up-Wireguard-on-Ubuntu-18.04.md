---
title: Setting up Wireguard on Ubuntu 18.04
date: "2019-11-05T22:12:03.284Z"
template: "post"
draft: false
slug: "/posts/Setting-up-Wireguard-on-Ubuntu-18.04/"
category: "Azure"
tags:
  - "Wireguard"
  - "Ubuntu"
  - "VPN"
description: "Setting up Wireguard as my VPN on a Ubuntu server 18.04"
---

## Installation Software

Referencing the [installation page](https://www.wireguard.com/install/)

First, add the WireGuard PPA to your apt configuration. Press ENTER when prompted

```shell
sudo add-apt-repository ppa:wireguard/wireguard
```

Once the PPA has been added, update the local package index, then install WireGuard.

```shell
sudo apt-get update
sudo apt-get install wireguard
```

## Generate keys

Referencing the [Key Generation](https://www.wireguard.com/quickstart/#key-generation) page.

You will need a private and public key for each computer, the computers must have each other's public key on either end of the connection.

First create the private key with the below commands:

```shell
umask 077
wg genkey | tee privatekey
```

You can then derive your public key from your private key:

```shell
wg pubkey < privatekey > publickey
```

Or generate both keys with one line:

```shell
wg genkey | tee privatekey | wg pubkey > publickey
```

## Config

After I installed Wireguard, I did the following configurations

### Enable IPv4 forwarding

First intall the linux header

```shell
sudo apt install linux-headers-*$(*uname -r*)
```

then enable IPv4 forwarding

```shell
sysctl -w net.ipv4.ip_forward=1
```

To make the change permanant, insert or edit the following line in `/etc/sysctl.d/99-sysctl.conf`

```shell
net.ipv4.ip_forward = 1  
```

### Wireguard configuration

Edit `/etc/wireguard/wg0.conf`

```shell
[Interface]
Address = 10.0.0.1/24
SaveConfig = false
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o enp2s0 -j MASQUERADE; ip6tables -A FORWARD -i wg0 -j ACCEPT; ip6tables -t nat -A POSTROUTING -o enp2s0 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o enp2s0 -j MASQUERADE; ip6tables -D FORWARD -i wg0 -j ACCEPT; ip6tables -t nat -D POSTROUTING -o enp2s0 -j MASQUERADE
ListenPort = 51820
PrivateKey = 0DpTFbW/p8NAmZd8O9FolgPCgX7N+gVc2LhvmHMbL0A=
```

* **Address** defines the private IPv4 and IPv6 addresses for the computer in this network. Each peer in the VPN network should have a unique value for this field.
* **ListenPort** specifies which port WireGuard will use for incoming connections.
* **PostUp** and **PostDown** defines steps to be run after the interface is turned on or off, respectively. In this case, iptables is used to set Linux IP masquerade rules to allow all the clients to share the server’s IPv4 and IPv6 address. The rules will then be cleared once the tunnel is down.
* **SaveConfig** tells the configuration file to automatically update whenever a new peer is added while the service is running.

### Firewall

I use ufw on my server, thus use this line to open the **ListenPort** of wireguard.

```shell
sudo ufw allow 51820/udp
```

## Enable

Once configuration is finished, use `wg-quick` to quickly enable the wg0 interface.

```shell
wg-quick up wg0
```

To enable auto start on boot, use the following:

```shell
sudo systemctl enable wg-quick@wg0
```

## Utilities

### Testing

To view connection status, use `wg`

```shell
% sudo wg
interface: wg0
  public key: DJtk4qLbQSlmm7CkrtTksQ2kUrTCuqfby5RG8qFZd1s=
  private key: (hidden)
  listening port: 51820

peer: qEUjlf3YlYvPFNe3akekJR+yf/zFWEweL/X3cUR+yFI=
  endpoint: 192.168.2.160:57547
  allowed ips: 10.0.0.9/32
  latest handshake: 50 seconds ago
  transfer: 15.50 MiB received, 508.70 MiB sent
  persistent keepalive: every 25 seconds

peer: RXMsmzLaCMUrMuK+uPtvvJi/FX/pwvTE6Fh1dtbtDFw=
  endpoint: 192.168.2.227:54696
  allowed ips: 10.0.0.4/32
  latest handshake: 58 seconds ago
  transfer: 3.12 KiB received, 10.18 KiB sent
  persistent keepalive: every 25 seconds
```

### Maintenance

To tear down the wg0 interface for reconfiguration or just to stop using it, use:

```shell
wg-quick down wg0
```

## Refernce

[How To Create a Point-To-Point VPN with WireGuard on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-create-a-point-to-point-vpn-with-wireguard-on-ubuntu-16-04)

[Set Up WireGuard VPN on Ubuntu](https://www.linode.com/docs/networking/vpn/set-up-wireguard-vpn-on-ubuntu/)

[Wireguard VPN: Typical Setup](https://www.ckn.io/blog/2017/11/14/wireguard-vpn-typical-setup/)
