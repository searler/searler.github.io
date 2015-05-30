---
layout: post
title:  "Injecting pigs into Kafka"
date:   2015-05-30 16:00:00
categories:  Kafka
---

Kafka is designed to move a stream of data, 
with an implicit assumption that the stream is continuous. 

There are use cases where boundaries need to be defined within the stream.
For example:

* End of day processing
* Change of message format
* Timeout a data source
* Compressed timestamps where message contains a delta from a preceding full timestamp.

Many of these use cases could be implemented by:

* calling out to a service from within the consumer implementation.
* explicitly calling the consumer with _out-of-band_ data.

Both of these require thread safe designs since the consumer will either be referencing data owned by another thread or
will be called by more than one thread. In either case, performance is adversely impacted. 

Alternatively, the boundary can be defined by a message produced into the existing Kafka stream.
The concept is something like the [pig](http://en.wikipedia.org/wiki/Pigging) used to seperate products in a pipeline. 

The implementation requires at least one (and ideally only one) pig message be delivered to each consumer.
This requires a pig message for each partition, ideally with one consumer per partition.

Fortunately, the [new Java producer](http://kafka.apache.org/082/javadoc/index.html) allows explicit specification of the partition for a message.

Note that mirrormaker does not currently provide the precise control over partitioning required for this implementation.








