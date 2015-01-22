---
layout: post
title:  "Akka Camel Consumer integration with Spark Streaming"
date:   2015-01-21 21:20:00
categories:  scala camel akka spark
---

The following code creates a Receiver that reads from stdin via
the camel [stream](http://camel.apache.org/stream.html) component.

The input is gathered for 5 seconds and then reduced to a single string,
written to stdout.

{% highlight scala %}
import org.apache.spark._
import org.apache.spark.SparkContext._
import org.apache.spark.storage._
import org.apache.spark.streaming._
import org.apache.spark.streaming.dstream._
import org.apache.spark.streaming.receiver._

import akka.actor.{ Actor, Props }
import akka.camel.CamelMessage
import akka.camel.Consumer

class Storer extends Actor with ActorHelper with Consumer {
  def endpointUri = "stream:in"

  def receive = {
    case cm: CamelMessage => store(cm.body)
  }
}

object CamelApp {
  def main(args: Array[String]) {
    val conf = new SparkConf(false)
      .setMaster("local[*]")

    val ssc = new StreamingContext(conf, Seconds(5))

    val actorStream = ssc.actorStream[String](Props[Storer], "storer")

    actorStream.reduce(_ + " " + _).foreachRDD {
      (rdd, time) =>
        rdd.foreach {
          s: String => println(s)
        }
    }

    ssc.start()

    ssc.awaitTermination()
  }
}
{% endhighlight %}








