---
layout: post
title:  "Understanding akka quarantine"
date:   2014-07-12 16:00:00
categories: akka 
---

The akka documentation for [remoting](http://doc.akka.io/docs/akka/snapshot/scala/remoting.html) describes a _quarantine_ state,
which requires the restart of the impacted ActorSystem. That appeared to rather inconvenient and has triggered much discussion in
the mailing lists, some of it rather heated. 

Further research located a [discussion](https://groups.google.com/d/msg/akka-user/rR8H4dcTRRo/wcf6yR8rMQsJ) and 
[ticket comment](https://www.assembla.com/spaces/akka/tickets/3741-make-quarantine-permanent?comment=415940363#comment:415940363). 
These provide both a design rationale and operations that will trigger the quarantine: 

* Remote DeathWatch
* Remote actor deployment

This is a generally acceptable trade-off.





