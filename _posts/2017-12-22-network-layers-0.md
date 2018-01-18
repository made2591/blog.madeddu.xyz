---
title: "A journey through the network - Introduction"
tags: [theory, network, iso/osi, tcp/ip, saga]
---

### A journey through the network - Introduction
During the last year I understood one thing: sooner or later everyone should review network notions, so I decided to start writing articles about network fundamentals. As a main source I would use [Computer Networks](https://www.amazon.it/gp/product/9332518742/ref=oh_aui_detailpage_o01_s00?ie=UTF8&psc=1) and [TCP/IP Illustrated](https://www.amazon.it/gp/product/9332535957/ref=oh_aui_detailpage_o02_s00?ie=UTF8&psc=1).

In this article, I will talk about two important network architectures: the __OSI__ reference model and the __TCP/IP__ reference model. These two model has opposite characteristics, in particular:
- the __ISO/OSI__ model protocols' are not used any more, but the model itself is actually quite general and still valid and the features discussed at each layer are still very important;
- the __TCP/IP__ model is not of much use but its protocols are widely used;

Let's start from the first!

### The __ISO/OSI__ reference model
The __ISO/OSI__ is the Open Systems Interconnection (__OSI__) reference model for connecting open systems, __based on a proposal__ originally developed by the International Standards Organization (__ISO__): __OSI__ (for short) consists of a stack of protocols through which the implementation complexity of a communication system for networking is reduced. The __OSI__ model has seven layers.

<p align="center"><img src="https://image.ibb.co/jpZKUm/isoosi.png" alt="perceptron" style="width: 100%; marker-top: -10px;"/></p>

<span style="color:#A04279; font-size: bold;">__NOTE__</span> the __OSI__ model itself is not a network architecture: it just tells what each layer should do. But the International Standards Organization (__ISO__) has also produced standards for all the OSI layers, although these are not part of the reference model itself. Each one has been published as a separate international standard. What is a layer?

#### Layers
A layer identifies a communication protocol of the same level. __ISO/OSI__ performs a communication by levels, i.e. given two nodes _A_ and _B_, the _n(th)_ level of node _A_ can exchange informations with the _n(th)_ level of node _B_, but __not__ with the others. Each level in transmission realizes the communication with the corresponding level on the transit nodes or recipients using the specification provided by the level immediately below. Thus, __ISO/OSI__ encapsulates level _n_ messages in messages of the _n-1_ level. So if _A_ has to send, for example, an e-mail to _B_, the application (level 7) of _A_ will propagate the message using the layer below (level _6_) which in turn will use the _interfaces_ of the lower layer, until to arrive at communication or transmission on the channel or physical transmission medium.

##### The physical layer - 1 of 7
The <span style="color:#A04279; font-size: bold;">physical layer</span> is concerned with transmitting raw bits over a communication channel of low-level networking equipment, such as some hubs, cabling, and repeaters. Examples of hardware in this layer are network adapters, repeaters, network hubs, modems, and fiber media converters. The design issues have to do with making sure that when one side sends a 1 bit it is received by the other side as a 1 bit, not as a 0 bit. Typical questions here are:
- what _electrical signals_ should be used to represent a 1 and a 0?
- how many _nanoseconds_ a bit last?
- whether transmission may proceed simultaneously _in both directions_? 
- how the initial connection is established?
- how it is torn down when both sides are finished?
- how many _pins_ the network connector has?
- what each pin is used for?

These design issues largely deal with mechanical, electrical, and timing interfaces, as well as the physical transmission medium, which lies below the physical layer.For instance, bit rate control is done at the physical layer. It may define transmission mode as simplex, half duplex, and full duplex. It defines the network topology as bus, mesh, or ring being some of the most common.

##### The data link layer - 2 of 7
The <span style="color:#A04279; font-size: bold;">data link layer</span> provides a node-to-node data transfer - a link between two directly connected nodes - free of undetected transmission errors. To make a list, it:
- _detects_ and possibly _corrects_ errors that may occur in the physical layer by masking the real errors so that the network layer does not see them. It accomplishes this task by having the sender break up the input data into __data frames__ (typically a few hundred or a few thousand bytes) and transmit the frames sequentially. If the service is reliable, the receiver confirms correct receipt of each frame by sending back an __acknowledgement__ frame;
- _defines_ the protocol to establish and terminate a connection between two physically connected devices;
- _defines_ the protocol to keep a fast transmitter from drowning a slow receiver in data.

The _IEEE 802_ standard divides the data link layer into two __sublayers__:
- Medium access control (__MAC__) sub-layer, responsible for controlling how devices in a network gain access to a medium and permission to transmit data. This layer resolves broadcast networks additional issues related to access control in a shared channel;
- Logical link control (_LLC_) sub-layer responsible for identifying and encapsulating network layer protocols, and controls error checking and frame synchronization.

<span style="color:#A04279; font-size: bold;">__NOTE__</span> there is a protocol belonging to this layer, named Point-to-Point Protocol (__PPP__), that can operate over several different physical layers, such as synchronous and asynchronous serial lines.

##### The network layer - 3 of 7
To define _where_ the network layer operates, let's first introduce two definition.

	A datagram is a basic transfer unit typically structured in two components, a header and a payload section. 

The header contains all the information sufficient for routing from the originating equipment to the destination without relying on prior exchanges between the equipment and the network: they may include source and destination addresses as well as a type field. The payload is the data to be transported. This process of nesting data payloads in a tagged header is called encapsulation.

	A network is a medium (physical) to which many nodes can be connected, on which every node has an address and which permits nodes connected to it to transfer messages to other nodes connected to it by merely providing the content of a message and the address of the destination node.

A _network_ is able find the way to deliver the message to the destination node, possibly routing it through intermediate nodes: if the message is too large to be transmitted from one node to another on the data link layer between those nodes, the network may implement message delivery by splitting the message into several fragments at one node, sending the fragments independently, and reassembling the fragments at another node. It may, but does not need to, report delivery errors. Let's move one step forward, introducing the network layer.

The <span style="color:#A04279; font-size: bold;">network layer</span> provides a way to transmit datagrams from one node to another connected in "different networks": it works at the subnet level. A key design issue is determining _how packets are routed from source to destination_. Routes can be based on static tables that are 'wired into' the network itself, and rarely changed, or more often they can be updated automatically to avoid failed components. They can also be determined at the start of each conversation, for example, a terminal session, such as a login to a remote machine. Finally, they can be highly dynamic, being determined aew for each packet to reflect the current network load.

When a packet has to travel from one network to another to get to its destination, many problems can arise. The addressing used by the second network may be different from that used by the first one. The second one may not accept the packet at all because it is too large. The protocols may differ, and so on. It is up to the network layer to overcome all these problems to allow heterogeneous net- works to be interconnected. In broadcast networks, the routing problem is simple, so the network layer is often thin or even nonexistent.

Sometimes too many packets are present in the network at the same time: handling congestion is also a responsibility of the network layer, in conjunction with higher layers that adapt the load they place on the network. More generally, the quality of service provided (delay, transit time, jitter, etc.) is also a network layer issue. Routing protocols, multicast group management, network-layer information and error, and network-layer address assignment are all layer-management protocols that belong to the network layer.

Message delivery at the network layer is not necessarily guaranteed to be reliable; a network layer protocol may provide reliable message delivery, but it need not do so. 

#### Chained vs end-to-end layers
In the layers seen until now, each protocols is between a machine and its immediate neighbors, and not between the ultimate source and destination machines, which may be separated by many routers. This is why layers 1 through 3, which are chained, are shown with a communication subnet boundary in the first figure. The layers from 4 through 7 are called end-to-end layers: they carries data all the way from the source to the destination. In other words, a program on the source machine carries on a conversation with a similar program on the destination machine, using the message headers and control messages.

##### The transport layer - 4 of 7
The <span style="color:#A04279; font-size: bold;">transport layer</span> accepts data from above it, splits them up into smaller units if need be, pass these to the network layer (lv 3), and ensure that the pieces all arrive correctly at the other end. Furthermore, all this must be done efficiently and in a way that isolates the upper layers from the inevitable changes in the hardware technology over the course of time. The transport layer also determines what type of service to provide to the session layer (lv 5), and, ultimately, to the users of the network. The most popular type of transport connection is an error-free point-to-point channel that delivers messages or bytes in the order in which they were sent.

An example of a transport-layer protocol is the Transmission Control Protocol (__TCP__), usually built on top of the Internet Protocol (__IP__). The transport layer controls the reliability of a given link through flow control, segmentation/desegmentation, and error control. Some protocols are state and connection-oriented this means that the transport layer can keep track of the segments and re-transmit those that fail. The transport layer also provides the acknowledgement of the successful data transmission and sends the next data if no errors occurred: it creates packets out of the message received from the application layer. Let's talk about the layers:

The __OSI__ defines 5 classes of connection-mode transport protocols: Class 0 contains no error recovery, and was designed for use on network layers that provide error-free connections. Class 4 is closest to __TCP__, although __TCP__ contains functions, such as the graceful close, which __OSI__ assigns to the session layer. In your opinion, Which class __UDP__ belong to?

Feature name | TP0 | TP1 | TP2 | TP3 | TP4
-------------|-----|-----|-----|-----|----
Connection-oriented network | Yes | Yes | Yes | Yes | Yes
Connectionless network | No | No | No | No | Yes
Concatenation and separation | No | Yes | Yes | Yes | Yes
Segmentation and reassembly | Yes | Yes | Yes | Yes | Yes
Error recovery | No | Yes | Yes | Yes | Yes
Reinitiate connection | No | Yes | No | Yes | No
Multiplexing / demultiplexing over single virtual circuit | No | No | Yes | Yes | Yes
Explicit flow control | No | No | Yes | Yes | Yes
Retransmission on timeout | No | No | No | No | Yes
Reliable transport service | No | Yes | No | Yes | Yes

##### The session layer - 5 of 7
The <span style="color:#A04279; font-size: bold;">session layer</span> allows users on different machines to establish sessions between them. Sessions offer various services, including dialog control (keeping track of whose turn it is to transmit), token management (preventing two parties from attempting the same critical operation simultaneously), and synchronization (checkpointing long transmissions to allow them to pick up from where they left off in the event of a crash and subsequent recovery). The OSI model made this layer responsible for graceful close of sessions, which is a property of the __TCP__ in Internet Protocol, and also for session checkpointing and recovery, which is not usually used in the Internet Protocol. The session layer is commonly implemented explicitly in application environments that use remote procedure calls.

##### The presentation layer - 6 of 7
The <span style="color:#A04279; font-size: bold;">presentation layer</span> is about the syntax and semantics of the information transmitted: it provides independence from data representation by translating between application and network formats. The presentation layer transforms data into the form that the application accepts.

##### The application layer - 7 of 7
The <span style="color:#A04279; font-size: bold;">application layer</span> contains a variety of protocols that are commonly needed by users. __HTTP__ (HyperText Transfer Protocol) is the most famous example: when a browser wants a Web page, it sends the name of the page it wants to the server hosting the page using HTTP. The server then sends the page back. Other application protocols are used for file transfer, electronic mail, and network news.

Let's talk about the __TCP/IP__ reference model!

### The __TCP/IP__ reference model
The __TCP/IP__ reference model was born from the _ARPANET_ network. _ARPANET_ was a research network sponsored by the U.S. Department of Defense, to connect hundreds of universities and government sites, using leased telephone lines. When satellite and radio networks were added later, the existing protocols had trouble interworking with them, so a new _reference_ architecture was needed. Thus, from nearly the beginning, the ability to connect multiple networks in a seamless way was one of the major design goals: the architecture designed to accomplish this goal later became known as the __TCP/IP__ reference model, after its two primary protocols.
Not only: in those years, DoD was scared by Soviet Union: thus, another major goal was that the network be _able to survive loss of subnet hardware_ without, of course, existing conversations being broken off. The requirement was to have a network able to mantain the connections intact as long as the source and destination machines were functioning, even if some of the machines or transmission lines in between were suddenly put out of operation.

#### Layers
Even in __TCP/IP__ encapsulation is used to provide abstraction of protocols and services. Encapsulation is usually aligned with the division of the protocol suite into layers of general functionality. Again, the layers of the protocol suite near the top are logically closer to the user application, while those near the bottom are logically closer to the physical transmission of the data. __TCP/IP__ is divided in 4 different levels

##### The link layer - 1 of 4
The <span style="color:#A04279; font-size: bold;">link layer</span> is the lowest component layer of the internet protocols: it is not really a layer at all, but rather an interface between hosts and transmission links, because __TCP/IP__ is designed to be hardware independent. As a result, __TCP/IP__ may be implemented on top of virtually any hardware networking technology.

The link layer is used to move packets between the Internet layer interfaces of two different hosts on the same link. This processes is usually done by the software device driver for the network card: this perform data link functions such as adding a packet header to prepare it for transmission, then actually transmit the frame over a physical medium. The __TCP/IP__ model includes specifications of translating the network addressing methods, used in the __IP__ to link layer addresses, such as media access control (__MAC__) addresses. All other aspects below that level, however, are implicitly assumed to exist in the link layer, but are not explicitly defined.

This is also the layer where packets may be selected to be sent over a _virtual private network_ or other _networking tunnel_. In this scenario, the link layer data may be considered application data which traverses another instantiation of the _IP_ stack for transmission or reception over another _IP_ connection. This is possible because the __TCP/IP__ model doesn't define a strict hierarchical encapsulation sequence.

The __TCP/IP__ model's link layer corresponds to the Open Systems Interconnection (__OSI__) model physical and data link layers (1 and 2) of the OSI model.

##### The internet layer - 2 of 4
The <span style="color:#A04279; font-size: bold;">internet layer</span> corresponding (quite) to the __OSI__ network layer and permit hosts to inject packets into any network and have them travel independently to the destination (potentially on a different network). They may even arrive in a completely different order than they were sent, in which case it is the job of higher layers to rearrange them, if in-order delivery is desired. The internet layer defines an official packet format and protocol called IP (Internet Protocol), plus a companion protocol called ICMP (Internet Control Message Protocol) that helps it function. The job of the internet layer is to deliver IP packets where they are supposed to go. Packet routing is clearly a major issue here, as is congestion (though IP has not proven effective at avoiding congestion).

##### The transport layer - 3 of 4
The <span style="color:#A04279; font-size: bold;">transport layer</span> is designed to allow peer entities on the source and destination hosts to carry on a conversation, just as in the __OSI__ transport layer. Two end-to-end transport protocols have been defined here: 
- the __TCP__ (Transmission Control Protocol) is a reliable connection-oriented protocol that allows a byte stream originating on one machine to be delivered without error on any other machine in the internet; it segments the incoming byte stream into discrete messages and passes each one on to the internet layer; at the destination, the receiving __TCP__ process reassembles the received messages into the output stream. __TCP__ also handles flow control to make sure a fast sender cannot swamp a slow receiver with more messages than it can handle;
- the __UDP__ (User Datagram Protocol) is an unreliable connectionless protocol for applications that do not want __TCP__'s sequencing or flow control and wish to provide their own. It is also widely used for one-shot, client-server-type request-reply queries and applications in which prompt delivery is more important than accurate delivery, such as transmitting speech or video;

##### The application layer - 4 of 4
The <span style="color:#A04279; font-size: bold;">application layer</span> contains all the higher-level protocols. The early ones included virtual terminal (_TELNET_), file transfer (_FTP_)and electronic mail (_SMTP_). Many other protocols have been added to these over the years.

### Conclusion
We talked a lot about the roles of each layers in __OSI__ reference models. Let's talk about 

\# | Layer        | OSI  | TCP                                   |
---|--------------|------|---------------------------------------|
 7 | Application  | [..] | Yes                                   |
 6 | Presentation | [..] | MIME / SSL / TLS / XDR                |
 5 | Session      | [..] | Sockets                               |
 4 | Transport    | [..] | TCP / UDP / SCTP / DCCP               |
 3 | Network      | [..] | IP / Ipsec / ICMP / IGMP / OSPF / RIP |
 2 | Data link    | MAC  | PPP / SBTV / SLIP                     |
 1 | Physical     | [..] |  /                                    |

<span style="color:#A04279; font-size: bold;">UPDATE 05/01/2018</span>: you can now read the second part of the Network Saga at the post [A journey through the network - Layer 1](https://made2591.github.io/posts/network-layers-1). Enjoy the reading!

Thank you everybody for reading!