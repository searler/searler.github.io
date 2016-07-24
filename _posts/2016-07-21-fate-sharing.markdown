---
layout: post
title:  "Lack of fate-sharing complicates distributed systems"
date:   2016-07-21 17:40:00
categories: distributed
---


This article [A Purpose-Built Global Network: Google's Move to SDN](http://cacm.acm.org/magazines/2016/3/198853-a-purpose-built-global-network/fulltext) makes reference to an interesting concept:[fate-sharing](https://en.wikipedia.org/wiki/Fate-sharing). The article and wikipedia entry both use the term in the context of network protocol/architecture design but the concept has broader applicability. 

The article does address the concept the context of a distributed system, referencing that problems that occur when a switch fails independently of its controller or vice-versa. 

Consider Leslie Lamport's famous [quote](http://research.microsoft.com/en-us/um/people/lamport/pubs/distributed-system.txt) "A distributed system is one in which the failure of a computer you didn't even know existed can render your own computer unusable". 

A non distributed system has only one machine and everything fails when it fails. There is no need to for failure detection or attempting if a slow response means the remote system has failed (or is merely slow). It also means that a reboot might fix things and is unlikely to make matters worse. 

The [STONITH](https://en.wikipedia.org/wiki/STONITH) principle attempts to reduce the impact of the problem by forcing a machine to a known state. 

The [Akka cluster](http://doc.akka.io/docs/akka/current/common/cluster.html) quarantines nodes to ensure they remain dead.






