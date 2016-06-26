---
layout: post
title:  "Misunderstanding actors"
date:   2016-06-26 14:20:00
categories:  actors 
---

The local [Java User Group](http://javamug.org) recently hosted a talk by [Dave Thomas](https://pragdave.me/) titled _Object-Orientation is Dead, Too_. The talk was interesting and attended by an overflow crowd. The topic, essentially functional programming is the future, was both interesting and well presented, despite being billed a _beta_ talk. 

David explicitly asked for feedback on this new talk, and some of the audience comments deserve further discussion. David described how actors (using Elixir/Erlang as an example) can be used to manage mutable state within an otherwise side effect free design. 

Much of the audience discussion then revolved around the issue of how to represent such design in a Java dominated environment. From my point of view, the short answer is Scala and Akka. 

One participant declared that each actor can be represented by a JVM instance! An Erlang/Elixir or Scala actor incurs a size overhead of a few hundred bytes and message passing only requires the addition of an object pointer into a queue, perhaps a few hundred machine cycles. They are cheap enough to be used as freely as objects are used in a traditional OO design. A JVM instance requires 10s to 100s of MBytes and requires serialization for message passing, i.e. 5 to 6 orders of magnitude more expensive. The JVM lifecycle involves the operating system and would then require a complex life cycle management solution (e.g. Mesos). The number of JVM instances would be limited to 100s, making it impossible to allocate one to each entity of interest in a non-trivial system. The participant might perhaps have been mislead by a comment that actors are like nano-services. Allocating a full JVM is de rigueur for SOA and micro-services, but the usage of the same word does not imply any commonality in design. 

Another participant thought that an actor could be implemented by a thread. The memory footprint is then approximately 1 Mbyte, only 3 orders of magnitude more expensive. The number of instances is limited to 1000s, which is still too small to allow an actor for every entity. A design that dedicates a thread per entity essentially predates Java 5, which introduced thread pools. Modern designs treat threads as a scarce and shared resource, that is allocated to an entity only when it actually needs to run on the CPU. A fully non-blocking design would only use as many threads are there are cores on which they can execute.
















