---
title: My homelab - 2023 edition
date: "2023-12-31T00:00:00.000Z"
template: "post"
draft: false
slug: "/posts/my-homelabe-2023-edition"
category: "Homelab"
tags:
  - "Homelab"
  - "Proxmox"
  - "Kubernets"
description: "How to install the helm-secrets plugin"
---

A Homelab is a constantly evolving thing. Here is a snapshot of my homelab by the end of 2023. I still have many changes in mind so 2024 will be a busy year!
![homelab diagram](./2023-12-31---My-homelab-2023/homelab-diagram.png)

## Hardware

### Router - Asus AC68U

This is the networking entry point to my home, connected to the AT&T modem. I have port-forwarding configured on it for specific services I want exposed to the internet. I turned off the WiFi radios to prevent interference with the access points.

### APs - Ubiquiti U6 pro x2

I have one access point (AP) per floor in my apartment. These WiFi 6 APs broadcast three networks: a dual-band 2.4/5 GHz main network, a 2.4 GHz IoT network, and a guest network. The APs are managed by a UniFi controller running locally on a Raspberry Pi.

### Raspberry Pi 3B+

I repurposed an old Raspberry Pi by installing Raspbian, using it to host lightweight applications that can be easily restarted by connecting or disconnecting power.

### NAS - Synology DS-720+

This NAS contains four 8TB hard drives in RAID 6 configuration, with 6GB RAM and two 1TB SSDs for cache. It's connected to a CyberPower CP1500PFCLCD UPS, configured to safely shut down during low battery events.

### Proxmox01

This is a custom-built PC featuring a 16-core Ryzen 5950X processor, 64GB of ECC DDR4 RAM, and a 2TB SSD for storage. It runs [Proxmox](https://www.proxmox.com/en/) for virtual machine management.

### Proxmox02

This is a refurbished Dell system currently running Proxmox, though I haven't finalized its intended purpose.

## Software

### Containerized Workloads

#### Pi-hole

Two [Pi-hole](https://pi-hole.net/) instances handle DNS queries for my home network: one on the Raspberry Pi and another in a Proxmox VM. The Asus router is configured to use both instances for DNS resolution. Ad blocking is currently disabled to avoid disrupting my roommates' browsing experience until I implement a simple way for them to toggle the blocking feature. The Pi-hole instances also manage custom domain names for local services.

#### Wireguard VPN

This is a [wg-easy](https://github.com/wg-easy/wg-easy) Docker container designed for simplified WireGuard setup. Clients connect via a custom domain name, with traffic port forwarded from the router.

### Kubernetes Cluster

My Kubernetes cluster is my personal lab for messing around with infrastructure-as-code (IaC). Since my homelab is constantly changing (and things inevitably break!), I wanted a way to rebuild the whole thing from scratch without too much manual work.

I used Terraform to spin up five Debian VMs, which I then turned into a high-availability Kubernetes cluster with three master and two worker nodes. I use Helmfile to manage my app deployments, so I can deploy or delete anything with a single command. For persistent storage, I'm using an NFS share on my Synology NAS, connected with the nfs-subdir-external-provisioner.

To make my services accessible with custom domain names, I'm using MetalLB and Traefik. Cert-manager and Let's Encrypt (using the DNS-01 challenge) automatically handle all my certificates. For monitoring, I've got the Prometheus/Grafana stack running, with node-exporter on each node.

I'm currently trying out Plex and Jellyfin for media serving.
