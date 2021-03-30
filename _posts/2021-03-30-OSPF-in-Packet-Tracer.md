---
layout: single
title: OSPF in Packet Tracer
date: 2021-03-30
classes: wide
header:
  teaser: /assets/images/OSPF/teaser.PNG
tags:
  - OSPF
  - Packet Tracer
  - Dynamic Routing
--- 

# Introduction

In this post we'll be doing a little bit of dynamic routing in [Packet Tracer](https://www.netacad.com/courses/packet-tracer). I'm going to assume that you already have Packet Tracer installed and have an account with NetAcad.

# What is OSPF?

Open Shortest Path First (OSPF) is a dynamic routing protocol that allows routers to forward packets based on the current conditions of the network. OSPF enabled routers communicate with each other to discover routes so there is no need for static configurations.

# Addressing 

| Device         | Interface     | IP          |
|:--------------:|:-------------:|:-----------:|
| laptop0        | fa0/0         | 192.168.0.2 |
| laptop1        | fa0/0         | 192.168.1.2 |
| router0        | gig0/0        | 192.168.0.1 |
| router0        | gig0/1        | 10.0.0.1    |
| router1        | gig0/0        | 192.168.1.1 |
| router1        | gig0/1        | 10.0.0.2    |

![](/assets/images/OSPF/addressing-table.PNG)

# Configuring the routers

## Initial Setup

Drag two 2911 routers into the screen

![](/assets/images/OSPF/drag-2911.PNG)

Connect the two via crossover cable, interface gig0/1 to gig0/1

![](/assets/images/OSPF/crossover.PNG)

## Interface gig0/1

Click on "router0", then click "CLI" and type the following commands to configure interface gig0/1.

```
en
conf t
int gig0/1
ip address 10.0.0.1 255.255.255.0
no sh
```

Click on "router1", then click "CLI" and type the following commands to configure interface gig0/1.

```
en
conf t
int gig0/1
ip address 10.0.0.2 255.255.255.0
no sh
```

![](/assets/images/OSPF/gig0-1-up.PNG)

## Interface gig0/0

Click on "router0", then click "CLI" and type the following commands to configure interface gig0/0.

```
en
conf t
int gig0/0
ip address 192.168.0.1 255.255.255.0
no sh
```

Click on "router1", then click "CLI" and type the following commands to configure interface gig0/0.

```
en
conf t
int gig0/0
ip address 192.168.1.1 255.255.255.0
no sh
```

![](/assets/images/OSPF/gig0-0-up.PNG)

## Enabling OSFP

Click on "router0", then click "CLI" and type the following commands to configure OSPF.

```
en
conf t
router ospf 100
network 10.0.0.0 0.0.0.255 area 0
network 192.168.0.0 0.0.0.255 area 0
```

Click on "router1", then click "CLI" and type the following commands to configure OSPF.

```
en
conf t
router ospf 100
network 10.0.0.0 0.0.0.255 area 0
network 192.168.1.0 0.0.0.255 area 0
```

![](/assets/images/OSPF/learning.PNG)

# Configuring laptops

## Initial Setup

Drag two laptops into the screen 

![](/assets/images/OSPF/laptops-in.PNG)

Connect "laptop0" from FastEthernet0 to gig0/0 on "router0" using a straight through cable

![](/assets/images/OSPF/laptops1-connected.PNG)

Connect "laptop1" from FastEthernet0 to gig0/0 on "router1" using a straight through cable

![](/assets/images/OSPF/laptops2-connected.PNG)

# Configuring IPs 

Click "laptop0" and then press "Config", set a static gateway of "192.168.0.1".

![](/assets/images/OSPF/laptops1-gateway.PNG)

Click "FastEthernet0" then set an IP Address of "192.168.0.2" and a Subnet Mask of "255.255.255.0".

![](/assets/images/OSPF/laptops1-fastetho.PNG)

Click "laptop1" and then press "Config", set a static gateway of "192.168.1.1".

![](/assets/images/OSPF/laptops2-gateway.PNG)

Click "FastEthernet0" then set an IP Address of "192.168.1.2" and a Subnet Mask of "255.255.255.0".

![](/assets/images/OSPF/laptops2-fastetho.PNG)

# Testing 

Attempt to ping from "192.168.0.2" to "192.168.1.2" using the "Desktop" then "Command Prompt" on "laptop0" using the command below.

```
ping -t 192.168.1.2
```

![](/assets/images/OSPF/ping-test.PNG)

On the routers you can run `show ip route ospf` to see the paths learned via OSPF.

![](/assets/images/OSPF/show-route.PNG)

