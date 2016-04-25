---
layout: post
title:  "Apache Spark Streaming backpressure"
date:   2016-04-24 21:20:00
categories:  scala spark
---

Spark streaming now provides automatic tuning of _spark.streaming.receiver.maxRate_ via the _spark.streaming.backpressure.enabled_ configuration flag. 

The configuration can still specify _spark.streaming.receiver.maxRate_, which then serves as the upper bound on the 
automatically computed value. That value needs to be reasonable, i.e. not much larger than the actual value. 

My experimentation indicates that an excessively large value (e.g. 10x) causes the streaming to start with an immediate overload,
from which it never recovers. 

