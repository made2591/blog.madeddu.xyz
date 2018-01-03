---
title: "A journey through the network - Layer 1"
tags: [theory, network, iso/osi, tcp/ip, saga]
---

### A journey through the network - Layer 0
Before the Christmas holidays, I wrote an article about the network: yes. The network is that part of computer science that is no longer considered fundamental as it should, and I must admit that I learn it every day at my expense: as an old friend always says to me "_the network is the concept on which everything is based, describes how the body works: after that, you can also become a gastroenterologist, but you will always need to know how the body works"_. I think he's right. As I was saying, I wrote an article about that: it's a sort of overview and technical / historical introduction on the ISO / OSI and TCP / IP protocols. For those who missed the part 1, [here](https://made2591.github.io/posts/network-layers-0) the link. Despite my lack of experience, I promised myself, as far as possible, with the little time available, to retrace the various levels of the network from a theoretical point of view without going into too much detail, also trying to identify the most used commands, understand the level at which they operate and the functioning of the parameters they make available. As a main source I use [Computer Networks](https://www.amazon.it/gp/product/9332518742/ref=oh_aui_detailpage_o01_s00?ie=UTF8&psc=1) and [TCP/IP Illustrated](https://www.amazon.it/gp/product/9332535957/ref=oh_aui_detailpage_o02_s00?ie=UTF8&psc=1). In this article, I will talk about layer 0, the lowest in the ISO / OSI stack. Enjoy the reading!

### Introduction
As Andrew Tanenbaum says in his book, "the physical layer is the foundation on which the network is built [..], so it is a good place to start our journey into networkland.". I agree, even if I would like to jump directly to higher network levels XD. I want to highlight that the physical layer I will talk about is defined in the __ISO/OSI__ standard, not in __TCP/IP__. __TCP/IP__ is designed to be hardware independent. As a result, __TCP/IP__ may be implemented on top of virtually any hardware networking technology. The lowest layer in __TCP/IP__ stack is the _link layer_: it is not really a layer at all, but rather an interface between hosts and transmission links used to move packets between the Internet layer interfaces of two different hosts on the same link. What I will talk about in the next paragraphs is the physical layer, defined in the __ISO/OSI__ standard, divided into four section:
- __Part 1__: Some theoretical analysis regarding data transmission and limits imposed by Universe on what can be sent over a channel;
- __Part 2__: The most important types of transmission media: guided (copper wire and fiber optics), wireless (terrestrial radio), and satellite;
- __Part 3__: The digital modulation, which is all about how analog signals are converted into digital bits and back again;
- __Part 4__: The multiplexing schemes aka _how multiple conversations can be put on the same transmission medium at the same time without interfering with one another_;

### Part 1 - Data trasmission
First, you need to know what is a <span style="color:#A04279; font-size: bold;">The Fourier transform</span>: you can find a complex explanation [here](https://en.wikipedia.org/wiki/Fourier_transform). For those afraid of math, including myself, I will try to gild the pill in the next lines. 

Let's start with waveform: virtually __everything__ in the world can be described via a waveform - a function of time, space or some other variable. For instance, sound waves, electromagnetic fields, the elevation of a hill versus location, the price of your favorite stock versus time, etc. The Fourier Transform gives us a unique and powerful way of _viewing_ these waveforms, because it proves [wait for it] an incredible fact, that deserves quotation:

    All waveforms, no matter what you scribble or observe in the universe, are actually just the sum of simple sinusoids of different frequencies.

This fact is amazing for lot of purpose, including the data trasmission in the physical layer. The Fourier Transform decomposes a waveform - basically any real world waveform - into sinusoids. Formally, the Fourier Transform is a _math tool_ that breaks a waveform into an _alternate_ representation, characterized by sine and cosines, and let us to represent the waveform itself in another way, as a sum of different fundamental frequencies, that combined together form the original waveform. Even if I don't want to go in depth with the math, consider a specific example: the transmission of the ASCII character ```b``` encoded in an 8-bit byte, represented by the bit pattern ```01100010```. The left-hand part of the figure below (a) shows the voltage output by the transmitting computer: the right part show the harmonic number. We know that the waveform in the left part, as stated before, is actually _the sum of simple sinusoids of different frequencies_. In the second part (b), there is the signal that results from a channel (a medium) that allows only the first harmonic (the fundamental, f) to pass through.

<p align="center"><img src="https://image.ibb.co/nctuuw/fourier_t.png" alt="perceptron" style="width: 100%; marker-top: -10px;"/></p>

Similarly, Fig. 2-1(c)–(e) show the spectra and reconstructed functions for higher-bandwidth channels. For digital transmission, the goal is to receive a signal with just enough fidelity to reconstruct the sequence of bits that was sent. Why not use a more accurate signal? Because, no transmission facility can transmit signals without losing some power in the process. The width of the frequency range transmitted without being strongly attenuated is called the __bandwidth__ and different medium have different bandwith. 

#### Bit rate, data rate and channels example
There is much confusion about bandwidth because it means different things to electrical engineers and to computer scientists. 
- To electrical engineers, (analog) bandwidth is a quantity measured in Hz (as we described below in the next example). 
- To computer scientists, (digital) bandwidth is the maximum data rate of a channel, a quantity measured in bits/sec. 
Let's make an example: given a bit rate (a sort of velocity $$v$$) mesured in $$\; \frac{bits}{sec} \;$$, the time required to send the $$8$$ bits (a sort of space quantity $$q$$) is given by

$$time = \frac{space}{velocity}, \; \; \rightarrow \; \; time = \frac{q}{v} seconds$$

In our example $$v = 1$$ bit at a time, so the _frequency_ of the first harmonic of this signal is $$v/8 \; Hz$$. Limiting the bandwidth limits the data (bits) rate (even for perfect channels completly noiseless). So what is the data rate? The data rate, as Tanenbaum says, "_is the end result of using the analog bandwidth of a physical channel for digital transmission_". If you want to know more about waveform, look [here](https://en.wikipedia.org/wiki/Waveform) and [here](https://en.wikipedia.org/wiki/Sine_wave).

##### Nyquist
Henry Nyquist, AT&T engineer, in 1924 realized that even a perfect channel has a finite transmission capacity. He derived an equation expressing the maximum data rate for a finite-bandwidth noiseless channel. 

<span style="color:#A04279; font-size: bold;">Difficult explanation</span>
Sampling is the first step in the analog-to-digital conversion process of a signal. It consists of taking samples (samples) from an analogue signal and continuing over time each $$\Delta t$$ seconds. 

<p align="center">
    <img src="https://upload.wikimedia.org/wikipedia/commons/c/ca/Analog_signal.png" alt="perceptron" style="width: 35%;  margin: 0 auto; marker-top: -10px;"/>
    <img src="https://upload.wikimedia.org/wikipedia/commons/6/6e/Sampled_signal.png" alt="perceptron" style="width: 35%; margin: 0 auto; marker-top: -10px;"/>
</p>

The value $$\Delta t$$ is called sampling interval, while $$f_s = \frac{1}{\Delta t}$$ is the sampling rate. The result is an analog signal in discrete time, which is then quantized, coded and made accessible to any digital computer. The Nyquist-Shannon theorem (or signal sampling theorem) states that, given an analog signal $$s(t)$$ whose frequency band is limited by the frequency $$f_M$$ and given $$n \in \mathbb{Z}$$, the signal $$s(t)$$ can be uniquely reconstructed from its samples $$s(n \Delta t)$$ taken at frequency $$f_s = \frac{1}{\Delta t}$$ if $$f_s > 2f_M$$ using the following formula:

$${\displaystyle{\displaystyle s(t) = \sum_{k=-\infty}^{+\infty}s(k \Delta t){\textrm{sinc}} \left ({\frac{t}{\Delta t}} -k\right) \; \forall t \in \mathbb{R}}}$$

expressed in terms of the normalized sync function[^sf]. What the f**k I said?! Don't know. 

<span style="color:#A04279; font-size: bold;">Simpler explanation</span>
The only thing you have to remember is that _if the signal consists of $$V$$ discrete levels (wait for example), Nyquist’s theorem states that the maximum data rate = $$2B * log_2(V) \; bits/sec$$. For example, a noiseless $$3 \; kHz$$ channel cannot transmit binary (i.e., two-level) signals at a rate exceeding 6000 bps, because $$2 * 3000 * log_2(2) \; bits/sec = 6000$$.

If random noise is present, the situation deteriorates rapidly. Let be $$S$$ the signal power, and $$N$$ the noise power. Than, the signal-to-noise ratio is $$S/N$$. Usually, this ratio is expressed on a log scale as the quantity $$10 * log_10(S/N)$$, because it can vary over a tremendous range. The units of this log scale are called __decibels__ (dB). Another genius, Shannon, derived an important result in this field: the maximum data rate or capacity of a noisy channel whose bandwidth is $$B \; Hz$$ and whose signal-to-noise ratio is $$S/N$$ is = $$B * log_2(1 + S/N) \; bits/sec$$

#### Media
Various physical media can be used for the actual transmission. Each one has its own niche in terms of bandwidth, delay, cost, and ease of installation and maintenance. Media are roughly grouped into guided media, such as copper wire and fiber optics, and unguided media, such as terrestrial wireless, satellite, and lasers through the air.

##### Guided Transmission Media
<span style="color:#A04279; font-size: bold;">Magnetic Media</span>
One of the most common ways to transport data from one computer to another is to write them onto magnetic tape or removable media (e.g., recordable DVDs), physically transport the tape or disks to the destination machine, and read them back in again.

| Pro | Cons |
--------------
| Bandwith | Transmission time | 

<span style="color:#A04279; font-size: bold;">Twisted Pairs</span>
One of the oldest and still most common transmission media is twisted pair. It consists of two insulated copper wires. The wires are twisted together in a helical form, just like a DNA molecule. Twisting is done because two parallel wires constitute a fine antenna. When the wires are twisted, the waves from different twists cancel out, so the wire radiates less effectively. A signal is usually carried as the difference in voltage between the two wires in the pair. This provides better immunity to external noise because the noise tends to affect both wires the same, leaving the differential unchanged. The most common application of the twisted pair is the telephone system. The bandwidth depends on the thickness of the wire and the distance traveled, but several megabits/sec can be achieved for a few kilometers in many cases.
Twisted-pair cabling comes in several varieties.
- __Unshielded Twisted Pair (UTP)__: UTP cables are not shielded. This entails a high degree of flexibility and resistance to efforts. They are widely used in ethernet networks.
<p align="center"><img src="https://image.ibb.co/nks4SG/Cavo_UTP_t.png" alt="perceptron" style="width: 40%; marker-top: -10px;"/></p>
- __Shielded Twisted Pair (STP)__: STP cables include a metal shield for each pair of cables. An example is that defined by IBM for its __token ring network__, but also that of the ANSI / TIA / EIA-568-A of the CAT5 shielded cable and later.
<p align="center"><img src="https://image.ibb.co/fv9jSG/Cavo_STP_t.png" alt="perceptron" style="width: 40%; marker-top: -10px;"/></p>
- __Screened Shielded Twisted Pair (S/STP)__: S/STP cables are STP cables further protected by a metal shield enclosing the entire cable; this further improves interference resistance. The latter must also be connected from both sides to the ground, to ensure greater protection against external waves.
<p align="center"><img src="https://image.ibb.co/drv2Ew/Cavo_S_STP_t.png" alt="perceptron" style="width: 40%; marker-top: -10px;"/></p>
- __Screened Unshielded Twisted Pair (S/UTP o FTP)__: S/UTP, also known as Foiled Twisted Pair (FTP) or Screened Foiled Twisted Pair (S/FTP), is an externally shielded UTP cable.
<p align="center"><img src="https://image.ibb.co/buPJnG/Cavo_S_UTP_t.png" alt="perceptron" style="width: 40%; marker-top: -10px;"/></p>

This construction methods are tipically groubed in several categories (source: wikipedia):

Name | Typical construction | Bandwidth | Applications | Notes |
-------------------------------------------------------------- |
Level 1 |  | 0.4 MHz | Telephone and modem lines | Not described in EIA/TIA recommendations. Unsuitable for modern systems.[9] |
Level 2 |  | 4 MHz | Older terminal systems, e.g. IBM 3270 | Not described in EIA/TIA recommendations. Unsuitable for modern systems.[9] |
Cat 3 | UTP[10] | 16 MHz[10] | 10BASE-T and 100BASE-T4 Ethernet[10] | Described in EIA/TIA-568. Unsuitable for speeds above 16 Mbit/s. Now mainly for telephone cables[10] |
Cat 4 | UTP[10] | 20 MHz[10] | 16 Mbit/s[10] Token Ring | Not commonly used[10] |
Cat 5 | UTP[10] | 100 MHz[10] | 100BASE-TX & 1000BASE-T Ethernet[10] | Common for current LANs. Superseded by Cat5e, but most Cat5 cable meets Cat5e standards.[10] |
Cat 5e | UTP[10] | 100 MHz[10] | 100BASE-TX & 1000BASE-T Ethernet[10] | Enhanced Cat5. Common for current LANs. Same construction as Cat5, but with better testing standards.[10] |
Cat 6 | UTP[10] | 250 MHz[10] | 10GBASE-T Ethernet | ISO/IEC 11801 2nd Ed. (2002), ANSI/TIA 568-B.2-1. Most commonly installed cable in Finland according to the 2002 standard EN 50173-1. |
Cat 6A | F/UTP, U/FTP | 500 MHz | 10GBASE-T Ethernet | Adds cable shielding. ISO/IEC 11801 2nd Ed. Am. 2. (2008), ANSI/TIA-568-C.1 (2009) |
Cat 7 | S/FTP, F/FTP | 600 MHz | 10GBASE-T Ethernet or POTS/CATV/1000BASE-T over single cable | Fully shielded cable. ISO/IEC 11801 2nd Ed. (2002) |
Cat 7A | S/FTP, F/FTP | 1000 MHz | 10GBASE-T Ethernet or POTS/CATV/1000BASE-T over single cable | Uses all four pairs. ISO/IEC 11801 2nd Ed. Am. 2. (2008) |
Cat 8/8.1 | F/UTP, U/FTP | 1600–2000 MHz | 40GBASE-T Ethernet or POTS/CATV/1000BASE-T over single cable | In development (ANSI/TIA-568-C.2-1, ISO/IEC 11801 3rd Ed.) |
Cat 8.2 | S/FTP, F/FTP | 1600–2000 MHz | 40GBASE-T Ethernet or POTS/CATV/1000BASE-T over single cable | In development (ISO/IEC 11801 3rd Ed.)

<span style="color:#A04279; font-size: bold;">Coaxial Cable</span>
The coaxial cable has better shielding and greater bandwidth than unshielded twisted pairs, so it can span longer distances at higher speeds. Coaxial cables used to be widely used within the telephone system for long-distance lines but have now largely been replaced by fiber optics on longhaul routes. 

<span style="color:#A04279; font-size: bold;">Power Lines</span>
The use of power lines (electricity distribution) for data communication is an old idea. In recent years there has been renewed interest in high-rate communication over these lines, both inside the home as a LAN and outside the home for broadband Internet access. Using electrical wires inside the home is not so easy, because of appliances switch on and off and creating electrical noise, electrical properties of the wiring change from house to house: international standards are actively under development because many products use various proprietary standards for power-line networking, allowing transport to a few hundred megabits.

<span style="color:#A04279; font-size: bold;">Fiber Optics</span>
The achievable bandwidth with fiber technology is 50 Tbps: we are nowhere near reaching these limits. The current practical limit of around 100 Gbps is due to our inability to convert between electrical and optical signals any faster. To build higher-capacity links, many channels are simply carried in parallel over a single fiber. However, the cost to install fiber over the last mile to reach consumers and bypass the low bandwidth of wires and limited availability of spectrum is tremendous. It also costs more energy to move bits than to compute. 
An optical transmission system has three key components: the light source, the transmission medium, and the detector. Conventionally, a pulse of light indicates a 1 bit and the absence of light indicates a 0 bit. The transmission medium is an ultra-thin fiber of glass. The detector generates an electrical pulse when light falls on it. By attaching a light source to one end of an optical fiber and a detector to the other, we have a unidirectional data transmission system that accepts an electrical signal, converts and transmits it by light pulses, and then reconverts the output to an electrical signal at the receiving end.
Thank you everybody for reading!

[^sf]: In mathematics, physics and engineering, $$sinc(x)$$ denotes the cardinal sine function or [sync function](https://en.wikipedia.org/wiki/Sinc_function)