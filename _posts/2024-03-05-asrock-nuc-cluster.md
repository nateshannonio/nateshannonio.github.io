---
title: "AsRock BOX / NUC Kubernetes Cluster"
date: 2024-03-05T02:00:00-04:00
categories:
  - blog
  - homelab
  - kubernetes
tags:
  - initial
  - homelab
  - kubernetes
---

My Homelab Docker/Kubernetes adventure has morphed over the years. 

It started with a single host running a few services using docker-compose.

Then it became some Rock64 SBCs... Then I came across some AsRock BOX / NUCs for cheap. I started with a two node cluster (yeah I know, not 3 nodes, but I was broke), and ran k3s as baremetal.

Overtime I ran into issues as I was learning, and would have to tear down my cluster and services leading to 'downtime'. 

After some time, I ran into a really good deal with some newer AsRock BOX 1340p on [Newegg](https://www.newegg.com/asrock-nuc-box-1340p-d4), and picked up 3 for a brand new cluster.
Rather than dealing with baremetal K3s, I opted this time to run proxmox and virtualize some Ubuntu nodes which then I installed k3s and built my clusters around.

I opted for these AsRock nodes because I've had good luck with AsRock over the years and these had dual 2.5Gbe and thunderbolt4 ports.

The nodes were built with 32Gb DDR4 and 2TB NVMe. Prices over the last year have been pretty good for flash, so picked up enough to cover for a bit. Not much is actually stored on the OS, but I also was looking at setting up a TB4 Ceph cluster across the nodes...
That may be a very future project to copy what Scyto has created [here](https://gist.github.com/scyto/4c664734535da122f4ab2951b22b2085).


Racking the cluster -

I came across the [MyElectronics](https://www.myelectronics.nl/us/nuc-minipc-19-3u-rackmount-kit-for-1-12-nucs.html) 3u rack, and figured it was worth the test to see if it would fit the nodes.
Well.. It doesn't fit perfectly, but it atleast cleaned up the rack. Only one of the Vesa screws lined up with the Intel NUC trays provided.


Bootstrapping the nodes -
With proxmox running, I made use of the two 2.5gbe ports by creating a LACP bond in proxmox

![Network Settings](/assets/images/proxmox-network-settings.png)