---
layout: post
title: Coding Apatheia, a P2P Protocol
---

## The Genesis of an Idea

![P2P STORE](https://github.com/adrianobrito/adrianobrito.github.io/assets/697706/94899945-7850-485e-8bb2-58e0f1e73174)

As a software engineer, I've always been fascinated by the intricacies of network communication. However, it wasn't until I began studying the BitTorrent protocol that I truly understood the potential of peer-to-peer (P2P) applications. This exploration led me to the Kademlia protocol, a P2P distributed hash table for decentralized peer-to-peer computer networks. Reading the [Kademlia paper](https://pdos.csail.mit.edu/~petar/papers/maymounkov-kademlia-lncs.pdf) was a revelation, and it inspired me to embark on a journey of creating a new protocol over Kademlia.

The vision was clear: to create a protocol that would act as an event stream/message bus, completely implemented in P2P. This would enable communication between many peers without the need for a centralized server. We would rely solely on the P2P data of the hosts connected to the network. This protocol would serve as the foundation for other protocols and applications to be implemented over it. A private message chat? A customized routing protocol? Internet of Things applications? The possibilities were endless.

The Kademlia protocol is a distributed hash table for decentralized P2P computer networks. It specifies the structure of the network and the exchange of information through node lookups. Kademlia nodes communicate among themselves using UDP. A key feature of Kademlia is its XOR metric, which captures the notion of distance in the network and helps in locating the closest nodes to a given node.

P2P networks, unlike traditional client-server models, do not rely on a central server to facilitate communication. Instead, each node in the network acts as both a client and a server, contributing to the overall health and resilience of the network. This decentralization is one of the key strengths of P2P networks, making them robust against failures and censorship.

## The Challenges

![image](https://github.com/adrianobrito/adrianobrito.github.io/assets/697706/e57c38f7-1b9c-4a9a-965b-62d53f3df6ee)

The journey was not without its challenges. As I began coding, things started to get weird. It was a paradigm shift to understand that most P2P hosts around the world are UDP servers and clients at the same time. This was a departure from the traditional client-server protocols architecture based on TCP (like HTTP, SMTP, FTP) that we're so accustomed to.

Another significant challenge was the amount of effort spent on encoding/decoding byte arrays into data and vice versa. This was a crucial part of the process, and it was imperative to have unit tests to ensure that all these parsing operations were working as expected. This was evident in the development process of the two pull requests: [#15](https://github.com/apatheia-org/apatheia-network/pull/15) and [#18](https://github.com/apatheia-org/apatheia-network/pull/18). 

The first integration tests were also a heavy step. We had to delve deep into [Apache MINA docs](https://mina.apache.org/mina-project/documentation.html) to understand how to code pure P2P UDP communication without any session in the middle. It was a challenging task, but we managed to make it work in the end by having a demo of the FindNode algorithm. All this coded in Scala. The choice of MINA as framework was done based on the fact that they provide a [very configurable](https://github.com/apatheia-org/apatheia-network/blob/main/src/main/scala/org/apatheia/network/client/impl/DefaultUDPClient.scala#L66) and [easy to setup](https://github.com/apatheia-org/apatheia-network/blob/main/src/main/scala/org/apatheia/network/server/impl/DefaultUDPServer.scala#L43) framework for UDP/TCP communication. 

## The Journey should go on

![image](https://github.com/adrianobrito/adrianobrito.github.io/assets/697706/270cbe09-564b-4091-adf2-7c28fb737968)

I am thrilled with how the protocol is evolving. However, it would be great to have collaborators to help. I actually don't know how to gather interested developers. All social media seems to be innefective for his. If you're reading this post [feel free to graB any issues](https://github.com/orgs/apatheia-org/projects/2/views/2) labelled as `good first issue`. I'm sure you will discover that the journey of coding a P2P protocol is an enlightening experience.

The world of P2P applications is vast and complex, but it's also incredibly exciting. As we continue to explore and innovate, I'm confident that we'll be able to create something truly remarkable. The future of Apatheia is bright. With the foundation of the protocol in place, we can now start building applications on top of it. Whether it's a private messaging chat, a customized routing protocol, or an Internet of Things application, the possibilities are endless. I'm excited to see where this journey takes us.

