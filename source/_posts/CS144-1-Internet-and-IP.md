---
title: 'CS144: 1. Internet and IP'
date: 2021-04-08 15:42:50
tags:
categories: [学习笔记, CS144]
description: Stanford课程CS144，介绍Internet和IP
---

# 1. Network Applications

- Read and write data over network
- Dominant model: **bidirectional**, **reliable** byte stream connection
  - One side reads what the other writes
  - Operates in both directions
  - Reliable (unless connection breaks)

Blow we will introduce three examples: the world wide web, BitTorrent and Skype

- **the world wide web**: A **client-server model**. A client opens a connection to a server and requests documents. The server responds with the  documents.
- **BitTorrent**: A **peer-to-peer model**. Swarms of clients open connections to each other to exchange pieces of data, creating a dense network of connections.
- **Skype**: A mix of the two models. When Skype clients can communicate directly, they do so in a peer-to-peer fashion. But sometimes the clients can’t  open connections directly, and so instead go through rendezvous or relay servers.

## 1.1 World Wide Web (HTTP)

<img src="https://maples31-blog.oss-cn-beijing.aliyuncs.com/img/image-20210409114932469.png" alt="image-20210409114932469" style="zoom:80%;" />

HTTP: Hypertext Transfer Protocol

GET, PUT, DELETE, INFO

200 OK response, 400 bad request

Model: Client sends a request by writing to the connection, the server reads the request, process it, and writes a response to the connection, which the client then reads.

## 1.2 BitTorrent

<img src="https://maples31-blog.oss-cn-beijing.aliyuncs.com/img/image-20210409115424186.png" alt="image-20210409115424186" style="zoom:80%;" />

BitTorrent is a program that allows people to share and exchange large files.

BitTorrent client requests documents from other clients.

A single client can request from many others in parallel, BitTorrent breaks files up into chunks of data called **pieces**. 

When a client downloads a complete piece from another client, it then tells other clients it has that piece so they can download it  too. These collections of collaborating clients are called **swarms**.

BitTorrent uses the exact same mechanism as the world wide web: **a reliable, bidirectional data stream**. When a client wants to download a file, it first has to find something called a **torrent file**.

 This torrent file describes some information about  the data file you want to download. It also tells BitTorrent about who the **tracker** is for that torrent.

A **tracker** is a node that  keeps track (hence the name) of what clients are members of the swarm. To join a torrent, your client contacts the tracker,  again, over HTTP, to request a list of other clients. Your client opens connections to some of these clients and starts requesting  pieces of the file. Those clients, in turn, can request pieces.

Furthermore, when a new client **joins the swarm**, it might tell this  new client to connect to your client. So rather than a single connection between a client and one server, you have a dense graph of connections between clients, **dynamically** exchanging data.

## 1.3 Skype

<img src="https://maples31-blog.oss-cn-beijing.aliyuncs.com/img/image-20210409120641088.png" alt="image-20210409120641088" style="zoom:80%;" />

**Skype**: the popular voice, chat, and video service. You have two personal computers requesting data from each other

**NAT** (Network Address Translator): If you’re  behind a NAT then you can open connections out to the Internet, but other nodes on the Internet can’t  easily open connections to you. Connections coming from the green side work fine, but connections coming from the red  side don't.

So the complication here is that if the client A wants to call the client B, it can’t open a connection. Skype  has to work around this.

<img src="https://maples31-blog.oss-cn-beijing.aliyuncs.com/img/image-20210409120932994.png" alt="image-20210409120932994" style="zoom:80%;" />

**Rendezvous server**: When you log into Skype, your client opens connections to a network of control servers.

**Process**: When client A calls client B, it sends a message to the rendezvous server. Since the server has an open connection to client B, it tells B that there’s a call request from A. The call dialog pops up on client B. If client B accepts the call, then it opens a connection to client A. Client A was trying to open a connection  to client B, but since B was behind a NAT, it couldn’t. So instead it sends a message to a computer that client B is already connected to, which then asks client  B to open a connection back to client A. Since client A isn’t behind a NAT, this connection can open normally.

This is called a **reverse connection** because it  reverses the expected direction for initiating the connection. Client A is trying to connect to client B, but instead client B opens a connection to client A.

<img src="https://maples31-blog.oss-cn-beijing.aliyuncs.com/img/image-20210409121246937.png" alt="image-20210409121246937" style="zoom:80%;" />

What does Skype do if both clients are behind NATs?

<img src="https://maples31-blog.oss-cn-beijing.aliyuncs.com/img/image-20210409121357730.png" alt="image-20210409121357730" style="zoom:80%;" />

**Relay**: A kind of server. Relays can’t be  behind NATs.

If both client A and client B are behind NATs, then the communicate through a  relay. They both open connections to the relay. When client A sends data, the relay forwards  it to client B through the connection that B opened. Similarly, when client B sends data, the  relay forwards it to client A through the connection client A opened.

# 2. The 4 Layer Internet Model

<img src="https://maples31-blog.oss-cn-beijing.aliyuncs.com/img/image-20210409122652010.png" alt="image-20210409122652010" style="zoom:80%;" />

Each layer  has a different responsibility,  with each layer building a  service on top of the one below,  all the way to the top where we  have the bi-directional reliable  byte stream communication  between applications.

## 2.1 Link Layer

The Internet is made up of end-hosts, links and routers. Data is  delivered hop-by-hop over each link in turn.

A **packet** consists of the  data we want to be delivered,  along with a header that tells the  network where the packet is to  be delivered, where it came  from and so on.

The Link Layer's **job**: carry the data over one link at a time.

**Examples**: Ethernet, WIFI

## 2.2 Network Layer

<img src="https://maples31-blog.oss-cn-beijing.aliyuncs.com/img/image-20210409123615601.png" alt="image-20210409123615601" style="zoom:80%;" />

The network layer's **job**: deliver packets end-to-end across the Internet from the source to the destination.

**Packet**: A **self-contained** collection of data, plus a header that describes what the data is, where it is going and where it came from.

Network layer packets are called **datagrams**. They consist of some data and a head containing the "To" and "From" addresses.

### Process

- The Network hands the datagram to the Link Layer below, telling it to send the datagram over the first link. (The Link layer is providing a **service** to the Network Layer.)

- At the other end of the link is a  **router**. The Link Layer of the  router accepts the datagram  from the link, and hands it up to  the Network Layer in the router. The Network Layer on the  router examines the destination  address of the datagram, and is  responsible for routing the  datagram one hop at a time towards its eventual destination.  It does this by sending to the  Link Layer again, to carry it  over the next link. And so on  until it reaches the Network  Layer at the destination.

### Separation

The Network Layer  does not need to concern itself  with *how* the Link Layer  sends the datagram over the link. 

This **separation** of concerns between  the Network Layer and the Link  Layer allows each to focus on  its job, without worrying about  how the other layer works. It also means that a single  Network Layer has a **common way** to talk to many different  Link Layers by simply handing them datagrams to send. This  separation of concerns is made  possibly by the **modularity** of  each layer and a common well-defined API to the layer below.

### Special

We must use the Internet protocol (IP)

- IP **makes a best-effort attempt** to deliver our datagrams to the other hand. But it **makes no promises**.
- IP datagrams can get lost, can be delivered out of order, and can be corrupted. There are **no guarantees**.

## 2.3 Transport Layer

**TCP** (Transmission Control Protocol): makes sure that data sent  by an application at one end of  the Internet is correctly  delivered (in the right order) to the application at the other end of the Internet.

**UDP** (User Datagram Protocol): just bundles up  application data and hands it to  the Network Layer for delivery  to the other end. UDP offers **no  delivery guarantees**.

## 2.4 Application Layer

HTTP (Hypertext Transfer Protocol)

## 2.5 ALL

Each layer communicates with its peer layer: each layer is only  communicating with the same  layer at the other end of the link  or Internet, without regard for  how the layer below gets the  data there.

<img src="https://maples31-blog.oss-cn-beijing.aliyuncs.com/img/image-20210409125350838.png" alt="image-20210409125350838" style="zoom:80%;" />

## 2.6 Other

### IP

<img src="https://maples31-blog.oss-cn-beijing.aliyuncs.com/img/image-20210409125453888.png" alt="image-20210409125453888" style="zoom:80%;" />

IP: the thin waist of the Internet

IP is the only choice if we want to use the Internet.

### OSI Model

<img src="https://maples31-blog.oss-cn-beijing.aliyuncs.com/img/image-20210409125603400.png" alt="image-20210409125603400" style="zoom:80%;" />

7-layer Open Systems Interconnection model --- OSI model

- Physical Layer: defined things like the voltage levels on a cable, or the physical dimensions of a connector
- Presentation Layer: provide services that allow communicating applications to interpret the meaning of data exchanged
  - data compression
  - data encryption (self-explanatory)
  - data description (data format)
- Session Layer: provides for delimiting and synchronization of data exchange
  - build a checkpointing
  - recovery scheme

# 3. The IP Service

## 3.1 The Internet Protocol

<img src="https://maples31-blog.oss-cn-beijing.aliyuncs.com/img/image-20210409162154225.png" alt="image-20210409162154225" style="zoom:80%;" />

**IP datagrams** consist of a header and some data.

When the transport layer has data to send, it hands a **Transport segment** to the Network layer below. The network layer puts the transport segments inside a new **IP datagram**. IP's job is to deliver the datagram to the other end.

IP send the datagram to the Link Layer that puts it inside a **Link Frame**, such as an Ethernet packet and ships it off to the first router.

- Application layer: stream of data
- Transport layer: segments of data
- Network layer: packets of data

## 3.2 The IP Service Model

<img src="https://maples31-blog.oss-cn-beijing.aliyuncs.com/img/image-20210409162645925.png" alt="image-20210409162645925" style="zoom:80%;" />

The IP service sends **Datagrams** from end host to end host; it is **unreliable**, but makes a **best-effort** to deliver the datagrams. The network maintains **no per-flow state** associated with the datagrams.

> Datagram: The datagram is packet that is routed individually through the network based on the information in its header. ---> **self-contained**
>
> IP DA --- IP address of the destination
>
> IP SA --- IP source address

## 3.3 Why is the IP service so simple?

- Simple, dumb, minimal: Faster, more streamlined and lower cost to build and maintain
- The end-to-end principle: Where possible, implement features in the end host.
- Allow a variety of reliable (or unreliable) services to be built on top.
- Works over any link layer: IP makes very few assumptions about the link layer below.

## 3.4 The Detail/features of the IP Service Model

1. Tries to prevent packets looping forever.
2. Will fragment packets if they are too long.
3. Uses a header checksum to reduce chances of delivering datagram to wrong destination.
4. Allows for new versions of IP (IPv4 --- 32 bit address; IPv6 --- 125 bit address)
5. Allows for new options to be added to header.

## 3.5 IPv4 Datagram

<img src="https://maples31-blog.oss-cn-beijing.aliyuncs.com/img/image-20210603232035399.png" alt="image-20210603232035399" style="zoom:80%;" />

