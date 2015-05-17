---
layout: post
title:  "Resource leak when using case classes with Spark UpdateStateByKey"
date:   2015-05-16 16:00:00
categories:  scala spark streaming
---

The Spark documentation describes how Scala case classes can be used to define
the data flowing through the system.

There are unfortunately is a leak of the reflection created classes needed for serialization,
especially when using updateStateByKey.
These [experiments](https://github.com/searler/SparkStreamingLeak) illustrate the difficulty. 










