---
layout: post
title:  "Reconnecting akka client"
date:   2016-11-25 15:12:00
categories:  scala akka
---

This post demonstrates a solution for the following use case:

1. Connect a server and print all the data received as a UTF8 String.
2. Retry connect every 4 seconds
3. Terminate on user command

The underlying requirement is gathering data from a Terminal Server,
where the serial port is exposed as TCP Server.

```src``` is created from BroadcastHub, so all instances of ```flow``` are referencing the same instance.
It provides the not-yet-completed Source so ```flow``` reads data from the server. 
It terminates ```flow``` when its underlying Promise is completed.

The Future exposed by ```flow._2``` completes when the remote server closes the connection, the server is not accessible or the ```src``` is completed. A special SENTINEL exception is used to detect the last case and
stop the retry loop.


{% highlight scala %}
  val (flag, src) = Source.maybe[Nothing].toMat(BroadcastHub.sink(2))(Keep.both).run()

  val SENTINEL = new IOException("stop")

  def make: Unit = {
    val tcpFlow = Tcp().outgoingConnection(new InetSocketAddress("127.0.0.1", 2558))
    val flow = tcpFlow.runWith(src, Sink.foreach(bs => println(bs.utf8String)))
    flow._2.onComplete(_ match {
      case Failure(SENTINEL) => //stop
      case x@_ => system.scheduler.scheduleOnce(4 seconds)(make)
    }
    )
  }

  make //start

  StdIn.readLine("Press enter to stop")
  flag.failure(SENTINEL)
{% endhighlight %}








