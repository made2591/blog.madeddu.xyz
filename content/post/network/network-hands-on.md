---
layout: post
title: "A journey through the network - Hands on (Part 1)"
date: 2019-02-18
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

### Prelude
A long long time ago ([*I can still remember*](https://www.youtube.com/watch?v=uxYpS_bVdhk) as Madonna sang) I started to wrote some posts about the network. For those who missed the previous posts, you can read [the introduction](https://made2591.github.io/posts/network-layers-0), [the physycal layer](https://made2591.github.io/posts/network-layers-1) and [the datalink layer](https://made2591.github.io/posts/network-layers-2) respectively. As a main source I use [Computer Networks](https://www.amazon.it/gp/product/9332518742/ref=oh_aui_detailpage_o01_s00?ie=UTF8&psc=1) and [TCP/IP Illustrated](https://www.amazon.it/gp/product/9332535957/ref=oh_aui_detailpage_o02_s00?ie=UTF8&psc=1). In the previous posts I had to go into details about how some parts of the physical layer works but - by going forward through the layers - concepts belonging to separate historical standards - OSI and IP - will intertwine and this entails some troubles from a _logical_ point of view. In this article, I will continue from there by going more practical with some hands-on to better understand this topics.

### Network stack: layers recap
As a recapp, this is a summary table with some mapping between what happens and to which levels things happening belongs to, with some example.

| OSI Ref. Layer No. | OSI Layer Equivalent               | TCP/IP Layer     | TCP/IP Protocol Examples                                                         |
| ------------------ | ---------------------------------- | ---------------- | -------------------------------------------------------------------------------- |
| 5,6,7              | Application, session, presentation | Application      | NFS, NIS, DNS, LDAP, telnet, ftp, rlogin, rsh, rcp, RIP, RDISC, SNMP, and others |
| 4                  | Transport                          | Transport        | TCP, UDP, SCTP                                                                   |
| 3                  | Network                            | Internet         | IPv4, IPv6, ICMP                                                                 |
| 2                  | Data link                          | Data link        | ARP, PPP, IEEE 802.2                                                             |
| 1                  | Physical                           | Physical network | Ethernet (IEEE 802.3), Token Ring, RS-232, FDDI, and others                      |

With respect to this table we are around level three. So the question we want to answer (spoiler: there will be no answer yet at the end of this article, unfortunately *it's a long trip*) is:

> What happen when you digit the URL `http://www.google.com` in your broswer?

Since I started moving bottom up for the network journey, to help find the right answer to this question I decided - as already said - to show some practical examples about how some network commands work. Because - as one of the mentor I had used (and use) to say - *in the beginning it was - and it's actually still today - all about network.*

### Agenda
The tools I selected to debug / study / understand networking foundamentals are almost all natively present in almost every operating systems since the beginning of time. The commands are almost in order of layers and I will go through some experiments to test their most important features. Most probably I will extend this list: right now, I thought about:

| Command Name | Description                                                                                                                              | Reference OSI Layer |
| ------------ | ---------------------------------------------------------------------------------------------------------------------------------------- | ------------------- |
| `ping`       | tool that uses the `ICMP` protocol's mandatory `ECHO\_REQUEST` datagram to elicit an `ICMP ECHO\_RESPONSE` from a host or gateway.       | 3                   |
| `iptables`   | tool to set up, maintain, and inspect the tables of IP packet filter rules in the os kernel.                                             | 3                   |
| `netcat`     | (also nc) simple Unix utility which reads and writes data across network connections, using `TCP` or `UDP` protocol.                     | 4                   |
| `curl`       | tool to transfer data from or to a server, using one of the supported protocols.                                                         | 4                   |
| `tcpdump`    | prints out a description of the contents of packets on a network interface that match the boolean expression.                            | 4                   |
| `tshark`     | network protocol analyzer.                                                                                                               | 4                   |
| `dig`        | (domain information groper) flexible tool for interrogating `DNS` name servers.                                                          | 5-7                 |
| `openssl`    | cryptography toolkit implementing the *Transport Layer Security* (`TLS v1`) network protocol, as well as related cryptography standards. | 6                   |
| `nmap`       | (network mapper) is an open source tool for network exploration and security auditing.                                                   | 3-7                 |

Let's start from `ping`.

#### ping
The most probably well known by eveyone `ping` it's actually a trivial command: it simply checks if your system can *reach* a host. To be more precise, it actually also provide some feedback regarding the *latency*. What does it mean *reaching an host*? Well, a system can reach another if it can estabilish a comunication with it throught network connectivity. There's no other way to think about reachability at this moment - remember that we are at 3rd layer thus **from a OSI layers perspective there's almost nothing under us**.
What about the latency? Let's give a brief definition.

##### About latency and simultaneity communications
The `latency` is the delay from input into a system to desired outcome. The term is understood slightly differently in various contexts and latency issues also vary from one system to another. Latency greatly affects how usable and enjoyable electronic and mechanical devices as well as communications are. The latency in communication is demonstrated in live transmissions from various points on the earth as the communication *hops* (a word who dealed with network should know) between a ground transmitter and a satellite and from a satellite to a receiver each take time. People connecting from distances to these live events can be seen to have to wait for responses. This latency is the wait time introduced by the signal travelling the geographical distance as well as over the various pieces of communications equipment. Even fiber optics are limited by more than just the speed of light, as the refractive index of the cable and all repeaters or amplifiers along their length introduce delays.

It could seem trivial but it is not at all: in fact, *latency is actually something related to strong concept about how the Universe itself works*. Even at infinitesimal distance, two systems cannot speak directly one to each other because - as Einstein stated as second principle in his Theory of Relativity (and it is demonstrated)[^simul]:

> The concept of simultaneity doesn't exist.

##### How to understand ping
To do my experiments, I created an EC2 instance on AWS but you can use any other system you have access to (at admin level, of course). I ensured during creation to assign to my instance a public IP and a security group with no rules. If you can setup a system like this, or you have another equivalent reachable host, open a shell and do

{{< highlight sh >}}
ping ec2-52-48-17-240.eu-west-1.compute.amazonaws.com
{{< / highlight >}}

or any other endpoint you want to test. If you created the instance like me you should see... 100% packet lost. Why? Because, the security group doesn't allow traffic. Thus, even sending a `ICMP` protocol's mandatory `ECHO\_REQUEST` datagram will be reject by default. So let's add a rule to that and see what happens. The needed EC2's associated security group inbound rule has to follow the schema:

- **Type**: Custom ICMP rule;
- **Protocol**: Echo Request;
- **Port**: N/A (could not be changed);
- **Source**: your choice (I would select My IP to be able to ping from the machine you are testing on);

First of all you should notice one important thing: **port range is not editable**. Of course, we are at layer 3. There are no ports. I said once again, because this is super important to understand:

> At layer 3 there are no ports.

`TCP` or `UDP` ports are defined in either layer 4 of the `OSI` model (or layer 3 of the `TCP/IP` model), both defined as the Transport layer. `OSI` layer 5 Session layer uses the ports defined in layer 4 to create sockets and sessions between communicating devices/programs/etc. Thus, at this point in time, we are not interested to them.

Since security rules are applied instantly, after adding the rule to allow `ping` by doing:

{{< highlight sh >}}
ping ec2-52-48-17-240.eu-west-1.compute.amazonaws.com
{{< / highlight >}}

you should see packets transmitted and respective times required. So let's **DESTROY the myth** of the statement: *if a host doesn't reply to ping, that means it's down*. **This is false**: actually, many server doesn't respond to ping.

Since this command is quite basic, I will no go through details about parameters: you can specify custom number of counts with `-c` (that means, stop after sending (and receiving) count `ECHO\_RESPONSE` packets), or `-i` wait seconds between sending each packet, etc. These are most probably the things you only have to know about `ping`.

#### iptables
The command `iptables` it's instead quite complex to describe: it lets you create rules to match network packets and accept, drop, modify, *whatever* them. It is actually used as a `Firewall` and `NAT`. What are these strangers?

##### Firewall
A Firewall is a software that monitor the network traffic. A firewall has a set of rules which are applied to each packet. The rules decide if a packet can pass, or whether it is discarded. Usually a firewall is placed between a network that is trusted, and one that is less trusted. When a large network needs to be protected, the firewall software often runs on a computer that does nothing else.

##### NAT
Network Address Translation (`NAT`) is a method of remapping one IP address space into another by modifying network address information in the IP header of packets while they are in transit across a traffic routing device. The technique was originally used as a shortcut to avoid the need to readdress every host when a network was moved. It has become a popular and essential tool in conserving global address space in the face of IPv4 address exhaustion. One Internet-routable IP address of a NAT gateway can be used for an entire private network. Actually, we all connect throught internet using a `NAT`. If you have more than one device inside you house, they can connect to internet even if your ISP provide to you only one Public IP. Now the question is: how is that possible?

In a typical configuration, a local network uses one of the designated private IP address subnets ([RFC 1918](https://tools.ietf.org/html/rfc1918)) - I have mine in 172.16.0.0, your router/access point provided by your ISP could have a default range in 192.168.0.0. A router on that network has a private address in that address space (have you ever search fro 192.168.1.1 in your browser?). What does it mean subnet?

##### Recap over subnetting
> Subnetting is the practice of dividing a network into two or more smaller networks. It increases routing efficiency, enhances the security of the network and reduces the size of the broadcast domain.

Thus, a subnet is a single small network created from a large network. How it is done? With masks! An IP address and a subnet mask both collectively provide a numeric identity to an interface. Both addresses are always used together. Without subnet mask, an IP address is an ambiguous address and without IP address a subnet mask is just a number. Both addresses are 32 bits in length. These bits are divided in four parts. Each part is known as octet and contains 8 bits. Octets are separated by periods and written in a sequence. Subnet mask assigns an individual bit for each bit of IP address. If IP bit belongs to network portion, assigned subnet mask bit will be turned on.

##### Subnet notation
There are three popular notations to write the IP address and Subnet mask: **decimal notation**, **binary notation** and **slash notation**. In decimal notation, a value range 1 to 255 represents a turned on bit while a value 0 (zero) represents a turned off bit. In binary notation, 1 (one) represents a turned on bit while 0 (zero) represents a turned off bit. The **slash notation** a slash (/) specifies the total number of the on bits in subnet mask that are written with the IP address instead of the full Subnet mask. Example are in the table.

| In Slash notation | In binary notation                                                             | In decimal notation              |
| ----------------- | ------------------------------------------------------------------------------ | -------------------------------- |
| 10.10.10.10/8     | 00001010.00001010.00001010.00001010 <br /> 11111111.00000000.00000000.00000000 | 10.10.10.10 <br /> 255.0.0.0     |
| 172.168.1.1/16    | 10101100.10101000.00000001.00000001 <br /> 11111111.11111111.00000000.00000000 | 172.168.1.1 <br /> 255.255.0.0   |
| 192.168.1.1/24    | 11000000.10101000.00000001.0000000 <br /> 111111111.11111111.11111111.00000000 | 192.168.1.1 <br /> 255.255.255.0 |

For instance, with /24 you can have up to 255 hosts inside 2^24 subnets. This is the reason most probably you have at home a local area in this range: because it's unlikely to find yourself having more than 255 devices connected in the same appartement, I guess.

##### Back to NAT
As we were saying, the router is also connected to the Internet with a public address assigned by an Internet Service Provider. As traffic passes from the local network to the Internet, the source address in each packet is translated on the fly from a private address (192 range, for instance) to the public address. The router tracks basic data about each active connection (particularly the destination address and port). When a reply returns to the router, it uses the connection tracking data it stored during the outbound phase to determine the private address on the internal network to which to forward the reply.

All IP packets have a source IP address and a destination IP address. Typically packets passing from the private network to the public network will have their source address modified, while packets passing from the public network back to the private network will have their destination address modified. To avoid ambiguity in how replies are translated, further modifications to the packets are required. The vast bulk of Internet traffic uses Transmission Control Protocol (`TCP`) or User Datagram Protocol (`UDP`). For these protocols the port numbers are changed so that the combination of IP address and port information on the returned packet can be unambiguously mapped to the corresponding private network destination.

#### How to understand iptables
To touch with your hands how iptables can act like a NAT we are gonna simualte a simple scenario locally, using our old friend Docker. This is the easiest way I found to test iptables and how it operates locally without breaking things in your host / need to spawn things in the cloud.

##### The scenario
The scenario is the following: you have an application that exposes a simple web server that outputs the IP addresses of the source and destination that makes request over it. This application is not reachable from outside world - in our scenario, from your host - but we have a proxy implemented with a few iptables rules that forwards all the traffic to our application and, from it, back to the requester. This proxy can dialogue both internally with our web server and externally with outside world - again, in the scenario your host will act like the outside world.

You can get all of what you need to go ahead with this experiments from [this gist](https://gist.github.com/made2591/a53db169ba6f52c24f20d76714e2e37b). Otherwise, just copy and paste the following content in a `docker-compose.yml` file and do a `docker compose up` in your host.

{{< highlight yaml >}}
version: "3"
services:
  proxy:
    container_name: proxy
    restart: always
    image: debian:jessie
    privileged: true
    ports:
      - "80:80"
    networks:
      - frontend
      - backend
    tty: true

  server:
    container_name: server
    restart: always
    image: yeasy/simple-web:latest
    privileged: true
    networks:
      - backend
    tty: true

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
{{< / highlight >}}

This docker will spawn two containers: the first is the [simple-web](https://hub.docker.com/r/yeasy/simple-web/) server described before, the second is a debian jessie image with nothing inside. What you should notice is that the server is attached *only to one network interface*, called `backend`. Instead, the proxy container is attached to both the `backend` and `frontend` network interfaces. This is a scenario that somehow simulate a private and a public subnet (two different subnets virtually not available and available, respectively, from outside world).

Actually, the container running the `debian:jessie` image doesn't act like *a real proxy* once is up and running. It only exposes the port 80 to your host. Thus, the first thing to test once you do `docker-compose up -d` is to test that only the proxy is reachable from it. How can you do it? With *curl* (part of this article, later in details). Even if the proxy doesn't expose anything, since the port 80 is mapped, if you *curl your localhost over 80* you should receive an empty reply. Let's do a curl over the proxy:

{{< highlight sh >}}
curl localhost 80
{{< / highlight >}}

Something like `Empty reply from server` should appear. This is a response: empty, but still a response. Our proxy is reachable! NOTE: if it is not, something is wrong and you should review your docker-compose.yaml. Not convinced yet? Before going ahead, let's stop the proxy for a moment and repeat our curl test again. From the same shell inside the host execute:

{{< highlight sh >}}
docker-compose stop proxy
curl localhost 80
{{< / highlight >}}

to stop the proxy and repeat the test. You should now receive a `Failed to connect to localhost port 80` that confirms that your proxy is not reachable since it's down. Let's start it again with:

{{< highlight sh >}}
docker-compose start proxy
{{< / highlight >}}

Now, we have to install `iptables` inside the proxy container and setup the rules. Our mission is to let a `curl` from host get the response from the `server` without exposing directly a port - just to test capabilities of iptables. From your host, get the container id of the proxy (first column is the id, last is the container name) by doing a `docker ps -a` - then enter inside the running container and install iptables. You can do it in one line command with:

{{< highlight sh >}}
docker exec -it <YOUR_PROXY_CONTAINER_ID> /bin/bash -c "apt-get update && apt-get install -y iptables"
{{< / highlight >}}

This will install `iptables` inside the container. You can even install directly from inside the docker, thus executing only the shell in interacting mode (remove from -c included to end from the command above), because actually it is needed for the next step. Before going ahead, let's see the server-container ip address to write our rules in the proxy. From your host do:

{{< highlight sh >}}
docker inspect <YOUR_SERVER_CONTAINER_ID>
{{< / highlight >}}

You should see a huge JSON describing the container, including the "Networks" interface and respective IP assigned to server. Keep it somewhere, you will need it in a while. From the host shell, enter inside the proxy container - if you are not already there in another shell - by doing:

{{< highlight sh >}}
docker exec -it <YOUR_PROXY_CONTAINER_ID> /bin/bash
{{< / highlight >}}

and then setup the iptables rules (you can copy and paste all together and press enter twice to export rules):

{{< highlight sh >}}
echo 1 > /proc/sys/net/ipv4/ip_forward
iptables -F
iptables -t nat -F
iptables -X
iptables -t nat -A PREROUTING -p tcp --dport 80 -j DNAT --to-destination <YOUR_SERVER_CONTAINER_IP>:80
iptables -t nat -A POSTROUTING -j MASQUERADE
{{< / highlight >}}

##### Explanation
The first line enables IP forwarding in the proxy container by telling that when it receives a packet not destined directly to it, to forward it to the destination (or, better, the next hop) instead of dropping it. This let the proxy works as a gateway to the specific destination. The other lines are iptables calls.

The `-F` parameter flush the selected chain (all the chains in the table if none is given). This is equivalent to deleting all the rules one by one. Of course, `-t nat -F` flush the rule of nat table (just to avoid noises in this experiment). `-X` is again a cleaning operation: it deletes the optional user-defined chain specified. If no argument is given, it will attempt to delete every non-builtin chain in the table. Before going ahead, remember that iptables work by - trivial, uhm? - tables XD. You should know that by default there are five tables in the kernel that contain sets of rules. Every tables has different chains that define exactly `when` - and `how` - iptables acts over packets that match the respective rules.

##### Tables
- **raw** is used only for configuring packets so that they are exempt from connection tracking;
- **filter** is the default table, and is where all the actions typically associated with a firewall take place;
- **nat** is used for network address translation (e.g. port forwarding);
- **mangle** is used for specialized packet alterations;
- **security** is used for Mandatory Access Control networking rules (e.g. SELinux);

I got from internet an ascii flow that should look almost like this:

{{< highlight sh >}}
                               XXXXXXXXXXXXXXXXXX
                             XXX     Network    XXX
                               XXXXXXXXXXXXXXXXXX
                                       +
                                       |
                                       v
 +-------------+              +------------------+
 |table: filter| <---+        | table: nat       |
 |chain: INPUT |     |        | chain: PREROUTING|
 +-----+-------+     |        +--------+---------+
       |             |                 |
       v             |                 v
 [local process]     |           ****************          +--------------+
       |             +---------+ Routing decision +------> |table: filter |
       v                         ****************          |chain: FORWARD|
****************                                           +------+-------+
Routing decision                                                  |
****************                                                  |
       |                                                          |
       v                        ****************                  |
+-------------+       +------>  Routing decision  <---------------+
|table: nat   |       |         ****************
|chain: OUTPUT|       |               +
+-----+-------+       |               |
      |               |               v
      v               |      +-------------------+
+--------------+      |      | table: nat        |
|table: filter | +----+      | chain: POSTROUTING|
|chain: OUTPUT |             +--------+----------+
+--------------+                      |
                                      v
                               XXXXXXXXXXXXXXXXXX
                             XXX    Network     XXX
                               XXXXXXXXXXXXXXXXXX
{{< / highlight >}}

Thus, the 4 command to iptables will act in the nat table over the PREROUTING chain, on packet received over TCP protocol on port 80. The destination `NAT` is specified using `-j DNAT '` and `--to-destination` option allows you to specify an address or range of IP addresses and optionally one or a range of ports (UDP and TCP protocols only). In our case, our server reachable by the proxy and not from outside. Let's see the rule alone once again:

{{< highlight sh >}}
iptables -t nat -A PREROUTING -p tcp --dport 80 -j DNAT --to-destination <YOUR_SERVER_CONTAINER_IP>:80
{{< / highlight >}}

Thus, this will change the destination address of web traffic in <YOUR_SERVER_CONTAINER_IP>:80. Unfortunately, it is not enought because: the server container will respond to the query but... the proxy would not be able to redirect this response back to the original requester. There is a specialized case of source NAT called **masquerading**: it should be used only if the IP addresses are assigned dynamically, such as in the case of external connection: in the case of static IP addresses instead use the already cited `SNAT`. This is the reason we created also the following rule:

{{< highlight sh >}}
iptables -t nat -A POSTROUTING -j MASQUERADE
{{< / highlight >}}

After applying the rules, if you open a browser to [http://localhost](http://localhost) you should see a webpage showing the requests counted. Ok, even if simple, this is was just an example to show a real case of how iptables works. There are so many things more but let's go ahead with the stack to not overcomplicate things.

#### netcat
Netcat it's a really nice tool: it lets you create `TCP` and `UPD` packets directly from your shell. Actually, it can do more: you can start listening over a specific port (as a real server) by simply running the command:

{{< highlight sh >}}
nc -l -p 9010 127.0.0.1
{{< / highlight >}}

If you run this in your machine everything you will send to 127.0.0.1:9010 will be printed out: let's try! In another shell, run the command:

{{< highlight sh >}}
echo "hello world" | nc 127.0.0.1 9010
{{< / highlight >}}

You will see hello world in the first shell in which you run nc as a listener. If two host are on the same network, you can actually use netcat to *send a file from a system to another*: this is as simple as replacing the 127.0.0.1 with the receiver local IP (in the sender) and redirecting the ouput (in the listener, or receiver, or server) to a file by adding the redirection ` > received_file` after the listener command. You can try easily how it works between two debian containers sharing the same network interface - by modify the docker-compose - or even try directly with two laptops inside your local network.

#### curl
The `curl` lets you make HTTP requests - in a more advanced way with respect to netcat. Let's modify our docker-compose by adding direct expose to the service server (add port expose as specified in the proxy service) and run the server alone by doing `docker-compose up server`. Got into a shell and run `curl 127.0.0.1 80` to verify that you can see the same response you first saw in the browser.

Some common and useful parameters you can pass to curl are:

- `-i`: it shows the response headers: let's try it with our server (from docker compose). Got into a shell and run `curl -i 127.0.0.1 80`. In addiction to response, you will also see the response HEADER, something like:

{{< highlight sh >}}
HTTP/1.0 200 OK
Server: SimpleHTTP/0.6 Python/2.7.14
Date: Tue, 05 Mar 2019 21:12:33 GMT
Content-type: text/html
Content-Length: 351
Last-Modified: Tue, 05 Mar 2019 21:12:33 GMT
{{< / highlight >}}

- `-I`: it will show only the response headers shown below by doing a `HEAD` request. This is very usefull if you want for instance to try availability of object somewhere without *paying the costs* of a real `GET` request. Let's say you have an object in S3 and you want to actively test if it is reachable or not. `HEAD` request are free of charge and if you get a status code 200, you know that the object you are looking for is reachable (or at least you know you have to look into it).
- `-X POST/GET/DELETE/PUT/ETC` will send the respective HTTP request. It is often use to test api together with standard GET requests (usually not explicited) with something like `curl --header "Content-Type: application/json" www.google.com`.
- `-L` follow the redirects (`3**` status) - useful when you are working with services authentication (it happened to me I add to test external 3 step auth with OpenID, etc).
- `-v` it's kind of verbose mode; you can of course combine together these parameters together;
- `-k` don't verify ssl certificates - usefull for some local development or if you are below not trusted corporate proxy (believe me or not it happened to me);
- `-k` don't verify ssl certificates - usefull for some local development or if you are below not trusted corporate proxy (believe me or not it happened to me);
- `--data` lets you send data. Unfortunately the simple server of iptables example doesn't work over POST request but ehy! there's the magical netcat that can do it for us. Open a shell and run `nc -l -p 9010 127.0.0.1`, then from another shell try running `curl -X POST http://127.0.0.1:9010 --data '{"hello": "world"}'` and you will se the JSON in output in netcat shell. If you run `curl -X POST http://127.0.0.1:9010 --data @helloworld.json` with `helloworld.json` a JSON a file with the JSON string `{"hello": "world"}` inside, it will work the same way by reading directly from the file. I used already this tecnique in the past for using Mapping APIs from ElasticSearch to index the files in my NAS (have a look [here](https://madeddu.xyz/posts/elasticnas/)).

### Conclusion
We arrived at almost the half of the list provided in the beginning: since this topic is huge, I will continue in another blog post asap with some cool and simple experiments with the remaining commands. You can play around with the gist and do some experiments to discover even more parameters and if you have some insights about usefull tips, don't hesitate put them in the comments.

The next time I will continue the experiments from `tcpdump`. Stay tuned and thank you everybody for reading!

[^simul]: If you want to know more about, follow [this link](https://en.wikipedia.org/wiki/Relativity_of_simultaneity).