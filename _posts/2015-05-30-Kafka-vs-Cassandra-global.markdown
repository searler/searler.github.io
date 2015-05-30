---
layout: post
title:  "Kafka and Cassandra costs for a multi data center deployment"
date:   2015-05-30 16:36:00
categories:  Kafka Cassandra data centers
---

Assuming 3 fold data replication.

A minimum Kafka deployment is essentially 3 Kafka and 3 Zookeeper nodes, for a total of 6 nodes.
A N data center design with aggregation then requires 6 * (N + 1) nodes. 

A minimum Cassandra deployment is 3 nodes, with an N data center design requiring 3 * N nodes. 

It would generally be expected that Zookeeper nodes require less hardware than either Kafka or Cassandra data nodes and 
that Kafka nodes require less hardware than Cassandra nodes. The Kafka system might thus cost less than the Cassandra system.

The Cassandra design is symmetrical while the Kafka design is a tree.
In other words, Cassandra clients could be distributed across all the data centers while Kafka clients can only reference the single
aggregation cluster.











