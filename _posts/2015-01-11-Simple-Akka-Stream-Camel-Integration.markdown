---
layout: post
title:  "Simple Akka Stream and Camel integration"
date:   2015-01-11 16:20:00
categories:  scala akka camel reactive
---

A trivial integration of [Akka Streams](http://doc.akka.io/docs/akka-stream-and-http-experimental/) with [Camel](http://camel.apache.org/),
intermediated by [Akka Camel](http://doc.akka.io/docs/akka/snapshot/scala/camel.html).

The code reads strings from stdin and writes the uppercased string to stdout.

This could be constrasted with a [direct](https://github.com/typesafehub/akka-contrib-extra/tree/master/src/main/scala/akka/contrib/stream) (and complete) implementation.

{% highlight scala %}
package sample.stream

import akka.stream.FlowMaterializer
import akka.stream.scaladsl.PublisherSource
import akka.stream.scaladsl.Source
import akka.stream.scaladsl.Sink
import akka.stream.actor.ActorSubscriber
import akka.stream.actor.ActorSubscriberMessage
import akka.stream.actor.OneByOneRequestStrategy
import akka.actor.{ ActorSystem, Props }

object CamelDriver extends App {
  import akka.camel.{ CamelMessage, Consumer, Producer }
  import akka.stream.actor.{ ActorPublisher }

  class CamelConsumer extends Consumer with ActorPublisher[String] {
    def endpointUri = "stream:in"

    import akka.stream.actor.ActorPublisherMessage._

    def receive = {
      case msg: CamelMessage =>
        msg.bodyAs[String] match {
          case "stop" => onComplete()
          case string if (totalDemand > 0) =>
            onNext(string)
        }
      case Request(_) => //ignored
      case Cancel =>
        context.stop(self)
    }
  }

  class CamelProducer extends Producer {
    def endpointUri = "stream:out"
  }

  class CamelSubscriber extends ActorSubscriber {
    import ActorSubscriberMessage._

    override val requestStrategy = OneByOneRequestStrategy

    val endPoint = context.actorOf(Props[CamelProducer])

    def receive = {
      case OnComplete             => system.shutdown()
      case OnNext(string: String) => endPoint ! string
    }
  }

  implicit val system = ActorSystem("some-system")
  implicit val materializer = FlowMaterializer()

  val source = Source[String](Props[CamelConsumer])
  val sink = Sink[String](Props[CamelSubscriber])

  source.map(_.toUpperCase).
    to(sink).
    run()
}
{% endhighlight %}








