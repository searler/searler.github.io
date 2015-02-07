---
layout: post
title:  "Using reactive extensions as a test tool"
date:   2015-02-07 16:20:00
categories:  reactive extensions
---

The July 2014 CACM has an interesting [article](http://cacm.acm.org/magazines/2014/7/176211-automated-qa-testing-at-electronic-arts/abstract) ,
which mentions that [Rx Extensions](https://msdn.microsoft.com/en-us/data/gg577609.aspx) can be used to implement test assertions.

Testing asynchronous systems is very difficult and there are a small number of supporting tools, e.g.:

* [Akka](http://doc.akka.io/docs/akka/snapshot/scala/testing.html)
* [Awaitility](https://code.google.com/p/awaitility/)


A Rx Extensions approach would be more declarative and hopefully easier to specify correctly. 
It is certainly best to avoid a design where the test harness needs its own tests to ensure
correctness!









