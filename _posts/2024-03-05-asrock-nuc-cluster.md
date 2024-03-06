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

I created a single Ubuntu VM on each proxmox node simply for handling a node failure. Other stuff may run as VMs, but for the time being, trying to only utilize kubernetes were possible.

![Nodes](/assets/images/proxmox-nodes.png)

From here, I utilized ansible to bootstrap all of the nodes with various settings and packages. (The repo is still private while I work through issues, but hope to make everything public soon.)

I recently came across this video from [Craft Computing](https://www.youtube.com/watch?v=IiwD8kcjD98) that talks about the BIGlittle with proxmox, and some updates that may improve performance. For the heck of it, I went ahead and did it on my nodes.

```
Microcode Install Instructions:
1) Install Proxmox 8.1
2) Add non-free-firmware to debian repo in sources.list
- Edit the /etc/apt/sources.list file. Add non-free-firmware to the 1st line so it looks like this---
- deb http://ftp.debian.org/debian bookworm main contrib non-free-firmware
3) Save Changes
4) #apt clean && apt update
5) #apt install intel-microcode
- The current version Debian has in the repo is 3.2023114.1~deb12u1
6) Reboot, and the microcode patch should apply automatically.
7) You can check what microcode you are running after reboot by grep 'stepping\|model\|microcode' /proc/cpuinfo
```

Once the cluster is bootstrapped and running k3s, I authenticated to it and checked to make sure everything is running.

I've been using Lens for some time now and recommend for a quick and easy way to view the cluster pretty quickly. [Lens](https://k8slens.dev/)

Now comes the fun part of seeing everything roll together in a short few steps.


Bootstrapping the cluster with Terraform -

Over the time I've spent with Terraform, I've made quite a few mistakes, but one setup I've started to like is the idea of base and composite modules. I'll explain more in a different topic, so for now, I'll just reference the modules I use.

I utilize terraform to bootstrap my kubernetes cluster with the bare minimum needed such as, [argo-cd](https://argo-cd.readthedocs.io/en/stable/), [onepassword-connect](https://developer.1password.com/docs/connect/), and [longhorn](https://longhorn.io/). 

ArgoCD - 

With Argocd I utilize the functionality called [ApplicationSets](https://argo-cd.readthedocs.io/en/stable/user-guide/application-set/) with the git generator. By defining an applicationSet object, Argocd will parse a git directory based on the definition, and create all the resources in the directory. This makes spawning all of the applications extremely easy. If I need a new application, I add the helm chart and values to the specific directory, and Argo handles the rest. 

OnePassword Connect -

With 1Password Connect, the biggest usecase is the syncing of secrets between 1password and kubernetes. This makes it so I'm not checking in any secrets to git, but it does have some gotchas because Connect utilizes deployment object annotations for generating and syncing the secrets. If your service doesn't utilize a deployment, Connect may not be best for you. Additionally, by using Terraform to bootstrap Connect, you need a local instance of Connect.

Why? 

With the Terraform I setup, I did not want to write the credentials or token into git, so I need Terraform to read those secrets from somewhere. Sure I can set as a env or tf var stored elsewhere, but thats another secret saved outside of the source of truth (1pass). So by having a local running docker instace of 1password Connect, terraform can use a data reference and pull the secrets from the container, and bootstrap the helm chart. Not perfect, but better than the alternative.

Longhorn -

With Longhorn, it's a distributed persistent storage in kubernetes. It allows me to define a persistent volume for a service, and have longhorn perform replication across the nodes for redudancy. Note: I also ran into some issues with Longhorn, and the fix was provided [Here](https://longhorn.io/kb/troubleshooting-volume-with-multipath/) 



Until next time.