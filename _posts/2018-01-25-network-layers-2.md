---
title: "A journey through the network - Layer 2"
tags: [theory, network, iso/osi, tcp/ip, saga]
---

### A journey through the network - Layer 2
A month ago I started to wrote some posts about the network. For those who missed the previous posts, [the introduction](https://made2591.github.io/posts/network-layers-0) and [the physycal layer](https://made2591.github.io/posts/network-layers-0). For the previous post I had to go into details about how some parts of the physical layer work but, by going forward with the layers, concepts belonging to separate historical standards - OSI and IP - will intertwine and this entails some troubles from a _logical_ point of view. I will try, as far as possible, to keep only the basic concepts of this layer: I also remember that this layer, together with the physical layer, are - at least in part - joined together in what is called the network access layer in the TCP / IP model.
As a main source I use [Computer Networks](https://www.amazon.it/gp/product/9332518742/ref=oh_aui_detailpage_o01_s00?ie=UTF8&psc=1) and [TCP/IP Illustrated](https://www.amazon.it/gp/product/9332535957/ref=oh_aui_detailpage_o02_s00?ie=UTF8&psc=1). In this article, I will talk about layer 1, the data link layer in the ISO / OSI stack. Enjoy the reading!

### Introduction
The data link layer uses the services of the physical layer to send and receive bits over communication channels.