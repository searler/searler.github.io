---
layout: post
title:  "Survey of Akka integration with Spark"
date:   2015-01-21 16:20:00
categories:  scala akka spark
---
Apache Spark now contains a [mechanism](http://spark.apache.org/docs/1.2.0/api/java/index.html?org/apache/spark/streaming/receiver/ActorHelper.html)
to integrate with Akka Actors, with further [documentation](http://spark.apache.org/docs/latest/streaming-custom-receivers.html). That material is 
fairly limited and does not provide much guidance. 

There is one [example](https://github.com/apache/spark/blob/master/examples/src/main/scala/org/apache/spark/examples/streaming/ActorWordCount.scala),
which is rather more complicated than might be first expected. 

Typesafe provide an [activator](https://typesafe.com/activator/template/spark-streaming-scala-akka) with 
[source](https://github.com/jaceklaskowski/spark-activator). This implementation directly references an internal Spark component 
[SparkEnv](http://spark.apache.org/docs/1.2.0/api/java/index.html?org/apache/spark/SparkEnv.html), whose documentation indicates that 
external usage is discouraged and it may be made private in a future release. The placement of SparkEnv within the Java API is a further
discouragement. 

Spark contains an official [zeromq](https://github.com/apache/spark/tree/master/external/zeromq)Receiver that uses ActorHelper,
without all the issues mentioned above. 

[SPARK-5267](https://issues.apache.org/jira/browse/SPARK-5267) calls for a standard [Camel](http://camel.apache.org/) 
implementation, without providing any details or source code. 

The [camel-spark](https://github.com/lachatak/camel-spark) project illustrates a Camel integration via 
[akka-camel](http://doc.akka.io/docs/akka/snapshot/scala/camel.html).

DataStax provide a [Cassandra Connector Demo](https://github.com/datastax/spark-cassandra-connector/blob/master/spark-cassandra-connector-demos/simple-demos/src/main/scala/com/datastax/spark/connector/demo/AkkaStreamingDemo.scala)

[AkkaUtils](https://github.com/apache/spark/blob/master/core/src/main/scala/org/apache/spark/util/AkkaUtils.scala)










