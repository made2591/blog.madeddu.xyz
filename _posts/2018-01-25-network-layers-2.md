---
title: "A journey through the network - Layer 2"
tags: [theory, network, iso/osi, tcp/ip, saga]
---

### A journey through the network - Layer 2
A month ago I started to wrote some posts about the network. For those who missed the previous posts, [the introduction](https://made2591.github.io/posts/network-layers-0) and [the physycal layer](https://made2591.github.io/posts/network-layers-0). For the previous post I had to go into details about how some parts of the physical layer work but, by going forward with the layers, concepts belonging to separate historical standards - OSI and IP - will intertwine and this entails some troubles from a _logical_ point of view. I will try, as far as possible, to keep only the basic concepts of this layer: I also remember that this layer, together with the physical layer, are - at least in part - joined together in what is called the network access layer in the TCP / IP model.
As a main source I use [Computer Networks](https://www.amazon.it/gp/product/9332518742/ref=oh_aui_detailpage_o01_s00?ie=UTF8&psc=1) and [TCP/IP Illustrated](https://www.amazon.it/gp/product/9332535957/ref=oh_aui_detailpage_o02_s00?ie=UTF8&psc=1). In this article, I will talk about layer 1, the data link layer in the ISO / OSI stack. Enjoy the reading!

### Introduction
The data link layer uses the services of the physical layer to send and receive bits over communication channels: there are several algorithms for achieving reliable, efficient communication of whole units of frames (or grouped bits - no more individual as in the previous layer) between two adjacent machines. What does it means adjacent? It means that the two machines are connected by a _communication channel that acts conceptually like a wire_, so that the bits are delivered in exactly the same order in which they are sent. Even if it might be simple, channels make errors, connections pass throught several machines, routers, etc... it's complicated. Let's start from the goals

#### Goals
- Providing a well-defined service interface to the network layer;
- Dealing with transmission errors;
- Regulating the flow of data so that slow receivers can comunicate with fast senders;

To guarantee this functionalities, the data link layer takes the packets it gets from the network layer and encapsulates them into frames for transmission: each frame is composed by a frame header, a payload field for holding the packet, and a frame trailer. It's really important to examine the principles of error control and flow control used in data link layer because they are used also by upper layers.

#### Offered services
The data link layer can be designed to offer various services. Following the Computer Networks book by A. Tanembaum:

| Service | Description | Use | Example(s) 
|---------|-------------|-----|------------
| Unacknowledged connectionless service | Consists of having the source machine send independent frames to the destination machine without having the destination machine acknowledge them. If errors happen, there are no attempt to detect and recover them | Appropriate when the error rate is very low, so recovery is left to higher layers, like real time traffic, voice | Ethernet 
| Acknowledged connectionless service | There are still no logical connections used, but each frame sent is individually acknowledged. If it has not arrived within a specified time interval, it can be sent again | This service is useful over unreliable channels, such as wireless systems | 802.11 (WiFi)
| Acknowledged connection-oriented service | With this service, the source and destination machines establish a connection before any data are transferred | Each frame sent over the connection is numbered, and the data link layer guarantees that each frame is sent exactly once, indeed received, in the right order | Satellite Channel, Long-Distance Telephone Circuit

When connection-oriented service is used, transfers go through three distinct phases. In the first phase, the connection is established by having both sides initialize variables and counters needed to keep track of which frames have been received and which ones have not. In the second phase, one or more frames are actually transmitted. In the third and final phase, the connection is released, freeing up the variables, buffers, and other resources used to maintain the connection.

#### Framing
Physical layer provide some sort of redundancy / correction error codes to transport bits over the channel: but data are not guaranteed to be sent over super noisy channels, so data link layer divides the stream of bits in frame, and add to each of them a checksum - with predefined algorithm. When a frame arrives at the destination, the checksum is recomputed: if it wrong, it can handle the error, normally sending back an error. Let's have a look at framing algorithms.

##### Algorithms
- Byte count: 
- Flag bytes with byte stuffing
- Flag bits with bit stuffing
- Physical layer coding violations










