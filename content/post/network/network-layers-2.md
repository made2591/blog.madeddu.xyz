---
layout: post
title: "A journey through the network - Layer 2"
date: 2018-01-25
categories:
- theory
- network
tags:
- theory
- network
- iso/osi
- tcp/ip
- saga
---

### A journey through the network - Layer 2
A month ago I started to wrote some posts about the network. For those who missed the previous posts, [the introduction](https://madeddu.xyz/posts/network-layers-0) and [the physycal layer](https://madeddu.xyz/posts/network-layers-0). For the previous post I had to go into details about how some parts of the physical layer work but, by going forward with the layers, concepts belonging to separate historical standards - OSI and IP - will intertwine and this entails some troubles from a _logical_ point of view. I will try, as far as possible, to keep only the basic concepts of this layer: I also remember that this layer, together with the physical layer, are - at least in part - joined together in what is called the network access layer in the TCP / IP model.
As a main source I use [Computer Networks](https://www.amazon.it/gp/product/9332518742/ref=oh_aui_detailpage_o01_s00?ie=UTF8&psc=1) and [TCP/IP Illustrated](https://www.amazon.it/gp/product/9332535957/ref=oh_aui_detailpage_o02_s00?ie=UTF8&psc=1). In this article, I will talk about layer 1, the data link layer in the ISO / OSI stack. Enjoy the reading!

<div class="img_container"><img src="https://i.imgur.com/vrvSyKA.jpg" style="width: 100%; marker-top: -10px;"/></div>

### Introduction
The data link layer uses the services of the physical layer to send and receive bits over communication channels: there are several algorithms for achieving reliable, efficient communication of whole units of frames (or grouped bits - no more individual as in the previous layer) between two adjacent machines. What does it means adjacent? It means that the two machines are connected by a _communication channel that acts conceptually like a wire_, so that the bits are delivered in exactly the same order in which they are sent. Even if it might be simple, channels make errors, connections pass throught several machines, routers, etc... it's complicated. Let's start from the goals.

#### Goals
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
- Byte count: not used anymore, consists of of set the number of byte in a field inside the header. The problem is that even the count can be garbled by an errors, and even if the destination thanks to the checksum knows that the frame is broken, it can't locate the correct start of the next frame.
- Flag bytes with __byte stuffing__: resynchronization is resolved using a flag byte that wraps a frame (both at the beginning and the ending). But what if the flag byte occurs in the data? One solution consists in escaping the flag byte with a an escape byte. What if the escape byte occurs? The escape byte is escaped. This scheme is a slight simplification of the one used in PPP (Point-to-Point Protocol).
- Flag bits with bit stuffing: framing can be also be done at the bit level. In the __HDLC__ (High-level Data Link Control) protocol each frame begins and ends with a special bit pattern, 01111110 or 0x7E in hexadecimal. This pattern is a flag byte. Whenever the sender's data link layer encounters five consecutive 1s in the data, it automatically stuffs a 0 bit into the outgoing bit stream. USB use this method, known as __bit stuffing__. When the receiver sees five consecutive incoming 1 bits, followed by a 0 bit, it automatically destuffs.
- Physical layer coding violations: this method consists in use redundancy signal (reversed) in physical layer to transport (indicate) the start and end of frames. It's a code violation to delimit frames.

A common pattern is to use a mix of these methods: for instance, Ethernet and 802.11 use a frame beginning with a well-defined pattern called a __preamble__. This pattern might be quite long (72 bits is typical for 802.11) to allow the receiver to prepare for an incoming packet. The preamble is then followed by a length (i.e., count) field in the header that is used to locate the end of the frame.

#### About errors
There are two basic strategies for dealing with errorsb: both of them add redundant information to the data that is sent.
- first strategy (using __error-correcting codes__ or __FEC__ - Forward Error Correction): include enough redundant information to enable the receiver to deduce what the transmitted data must have been.
- second strategy (using __error-detecting codes__): include enough redundant information to enable the receiver to deduce that an error has occurred.

Of course the best way to handle error on channel like fiber is to retransmit (they are faster). FEC is used on noisy channels because retransmissions are just as likely to be in error as the first transmission.

##### Error-Correcting Codes
These are the most famous error-correcting codes
- Hamming codes
- Binary convolutional codes
- Reed-Solomon codes
- Low-Density Parity Check codes

Before going on with details of each methods, let define a frame:

	A frame consists of m data (i.e., message) bits and r redundant (i.e. check) bits.

There are several ways to compute r codes:
__block code__: the r check bits are computed solely as a function of the m data bits with which they are associated, as though the m bits were looked up in a large table to find their corresponding r check bits.
__systematic code__: the m data bits are sent directly, along with the check bits, rather than being encoded themselves be- fore they are sent.
__linear code__: the r check bits are computed as a linear function - XOR or modulo 2 addition is a popular choice - of the m data bits.

Let the total length of a block be $$n$$ (i.e., $$n = m + r$$) - the $$(n, m)$$ code. An $$n$$-bit unit containing data and check bits is referred to as an $$n$$-bit codeword. The __code rate__, or simply rate, is the fraction of the codeword that carries information that is not redundant, or $$m/n$$. The rates used in practice vary widely. They might be 1/2 for a noisy channel, in which case half of the received information is redundant, or close to 1 for a high-quality channel, with only a small number of check bits added to a large message. Ok, but why codewords?

The number of bit positions in which two codewords differ is called the Hamming distance, and it is computed using the XOR of the two codewords and count the number of 1 bits in the result. Its significance is that if two codewords are a Hamming distance $$d$$ apart, it will require d single-bit errors to convert one into the other. Example: you need to design a code with $$m$$ message bits and $$r$$ check bits that will allow all single errors to be corrected. Each of the $$2^m$$ legal messages has $$n$$ illegal codewords at a distance of $1$ from it: each of them is formed by inverting each of the $$n$$ bits in the $$n$$-bit codeword formed from it. Each of the $$2^m$$ legal messages requires $$n + 1$$ bit patterns dedicated to it. Since the total number of bit patterns is $$2^n$$, we must have $$(n + 1) 2^m \leq 2^n$$. Using $$n = m + r$$, this requirement becomes $$(m + r + 1) \leq 2^r$$. Given $$m$$, this puts a lower limit on the number of check bits needed to correct single errors.

<span style="color:#A04279; font-size: bold;">__Hamming codes__</span> the bits of the codeword are numbered consecutively, starting with bit 1 at the left end, bit 2 to its immediate right, and so on. The bits that are powers of 2 (1, 2, 4, 8, 16, etc.) are check bits. The rest (3, 5, 6, 7, 9, etc.) are filled up with the m data bits. A bit is checked by just those check bits occurring in its expansion (e.g., bit 11 is checked by bits 1, 2, and 8). For more details over the other codes, look at chapter 4 of Tanembaum's book.

##### Error-Detecting Codes
There are three types of error-detecting codes:
- parity codes
- checksums codes
- cyclic redundancy checks (CRCs), also known as a polynomial code, are based upon treating bit strings as representations of polynomials with coefficients of 0 and 1 only.

#### About flow controls
When a node is transmitting to another node, even if the transmission is error free, the receiver may be unable to handle the frames as fast as they arrive and will lose some. To prevent this situation, two approaches are commonly used:
- __Feedback-based flow control__: the receiver sends back information to the sender giving it permission to send more data, or at least telling the sender how the receiver is doing.
- __Rate-based flow control__: the protocol has a built-in mechanism that limits the rate at which senders may transmit data, without using feedback from the receiver. This scheme is part of the transport layer.

### Conclusion
I don't want to go in details on sliding windows and MAC Protocol, because they are at really low levels: in the next chapter I will talk more about the network layer, concerned to get packets from the source all the way to the destination. Getting to the destination may require making many hops at intermediate routers along the way. This function clearly contrasts with that of the data link layer, which has the more modest goal of just moving frames from one end of a wire to the other. Thus, the network layer is the lowest layer that deals with end-to-end transmission: further, when the source and destination are in different networks, new problems occur. It is up to the network layer to deal with them. In next post I will talk about this layer primarily using the TCP/IP book so focusing mainly on the Internet standard and _its_ network layer protocol, IP.
