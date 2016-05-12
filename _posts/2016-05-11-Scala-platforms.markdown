---
layout: post
title:  "The multiple platforms for Scala"
date:   2016-05-10 21:20:00
categories:  scala 
---

Scala is a JVM language, with explicit design for good interoperability with Java. 

Scala.Net came to life in 2011 and died around 2013, despite support from EPFL.

[Scala.js](https://www.scala-js.org/) arrived in November 2013 and now has sufficient traction that important libraries have explicit support. I found this [story](http://underscore.io/blog/posts/2016/03/21/serverless-scale-summit.html) describing compiling to JS because on AWS Lamba node has faster startup than the JVM! It is no longer a refuge for people who hate having to use JS in the browser.

The latest addition is [scala.native](http://www.scala-native.org/), using LLVM to compile directly to native code. It is rather amusing to see the reference to "ahead-of-time" compiling, as if that is something unusual. 




