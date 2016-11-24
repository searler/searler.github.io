---
layout: post
title:  "Read-only TCP client in Akka Streams"
date:   2016-11-24 10:20:00
categories:  scala akka
---

This [stackoverflow question](http://stackoverflow.com/questions/35398852/reading-tcp-as-client-via-akka-stream) concerns how to implement a read-only TCP client using [Akka Streams](http://doc.akka.io/docs/akka/2.4/scala/stream/index.html).  

The key issue was the requirement that a Flow has both a Sink (to process the incoming bytes) and a Source. For this use case the Source does not generate any bytes, but must still be specified. 

```Source.empty``` would appear to satisfy the requirement, but it immediately completes the Flow. The above stackoverflow suggests using ```Source(List(ByteString.empty))```, but that also completes the Flow. 

We need an incomplete Source, which is most easily achieved using ```Source.maybe```, which never completes and explicitly specifies that no data will write to the Flow. 

The core code is then
{% highlight scala %}
  val clientFlow: Flow[ByteString, ByteString, Future[OutgoingConnection]] =
     Tcp().outgoingConnection(new InetSocketAddress("127.0.0.1", 2558))
  val result: (Promise[Option[Nothing]], Future[Done]) =
     clientFlow.runWith(Source.maybe, Sink.foreach(bs => println(bs.utf8String)))
{% endhighlight %}

The ```Promise[Option[Nothing]]``` provides a hook to cleanly terminate the Flow, using  
```result._1.complete(Try(None))```.

The ```Future[Done]``` provides a hook to determine the Flow has terminated, e.g.
```Await.ready(result._2, 100 seconds)```.









