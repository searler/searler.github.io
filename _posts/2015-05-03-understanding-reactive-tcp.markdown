---
layout: post
title:  "Understanding TCP behavior for Reactive Streams"
date:   2015-05-03 16:40:00
categories:  scala akka reactive stream
---

This [project](https://github.com/adamw/reactive-akka-pres) provides useful examples
with which to understand [Akka Reactive Streams](http://doc.akka.io/docs/akka-stream-and-http-experimental/1.0-RC2/scala.html)

Some of the code is a little surprising and required further analysis

The first example (simplified to the core functionality) is

{% highlight scala %}
class Experiment(receiverAddress: InetSocketAddress)(implicit val system: ActorSystem) {

  def run(): Unit = {
    implicit val mat = ActorFlowMaterializer()

    StreamTcp().bind(receiverAddress).runForeach { conn =>

       val receiveSink : Sink[ByteString,Unit] = conn.flow
          .transform(() => new ParseLinesStage("\n", 4000000))
          .to(Sink.foreach { line =>
             println(line)
           })
       
        Source.empty.to(receiveSink).run()
    }
  }
}

object Experiment extends App {
  implicit val system = ActorSystem()
  new Experiment(new InetSocketAddress("localhost", 9182)).run()
}
{% endhighlight %}

The surprising part is ```Source.empty.to(receiveSink).run()```.
What purpose is served by ```Source.empty```? 
The code fails to compile if it is omitted, but that does not explain the semantics.

Replacing ```Source.empty``` with ```Source.single(ByteString("hello"))``` 
The client now receives output _hello_

The ```conn.flow``` has type ```Flow[ByteString, ByteString, Unit]```, matching
our expectation that allows us to both read and write bytes. 

The ```receiveSink``` has type ```Sink[ByteString,Unit]```, indicating it consumes bytes.
Those bytes move _out_ through ```conn.flow``` to the client. 

```Source.empty``` is then simply a placeholder with which to complete ```receiveSink``` and 
allow the flow to receive data from the client.









