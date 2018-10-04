---
layout: post
title:  "Poor support for ZIO in Intellj"
date:   2018-10-03 21:20:00
categories:  scala zio
---


[ZIO](https://scalaz.github.io/scalaz-zio/) is an interesting FP effects library.
[Software Mill](https://github.com/softwaremill/akka-vs-scalaz) wrote an interesting comparison between ZIO and its 
competitors.

Exploring that code with Intellj indicated a potentially significant concern.

ZIO uses a bi-functor ```IO[E,A]``` to capture the types of both the error and value returned by the IO and requires
the kind projector. The resultant code is often too complex for Intellj to infer all the types, leading to false error indications.

The UsingZio.scala file illustrates the problem at line 23 : `crawlUrl(d2, link)` is flagged with

*Expression of type IO[Nothing, UsingZio.CrawlerData] doesn't conform to expected type G_[UsingZio.CrawlerData]*

The source of the problem lies on line 20 `links.foldM(data2)`. Adding the appropriate types resolves the false
error `links.foldM[IO[Nothing,?], CrawlerData](data2)`

The increased verbosity is irritating but not really a concern. A larger concern is the 
difficulty in determining the type that must be specified. 

This adversely impacts the usability of ZIO. The other libraries are not impact by this 
deficiency in the tooling and are thus easier to use. 






