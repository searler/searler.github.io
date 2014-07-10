---
layout: post
title:  "Recovery from cluster partition"
date:   2014-07-10 18:00:00
categories: akka scala cluster
---

The akka [documentation](http://doc.akka.io/docs/akka/snapshot/scala/cluster-usage.html) indicates that a node that 
has become disconnected from the cluster will only reconnect after the ActorSystem has been restarted. The
documentation is surprisingly silent on how that might be implemented, trade-offs and consequences.

Use the [akka cluster sample](https://typesafe.com/activator/template/akka-sample-cluster-scala) as a
starting point. Start the two backends on 2551 and 2552, and then start the frontend. Note that the
frontend does not reconnect if the two backends are stopped and then restarted. 

The code below modifies the frontend to shutdown the ActorSystem when the number of backends falls to zero
A callback then restarts the ActorSystem and associated actors once the ActorSystem shutdown is complete.



{% highlight scala %}
class TransformationFrontend extends Actor {

  var backends = IndexedSeq.empty[ActorRef]
  var jobCounter = 0

  def receive = {
    case job: TransformationJob if backends.isEmpty =>
      sender() ! JobFailed("Service unavailable, try again later", job)

    case job: TransformationJob =>
      jobCounter += 1
      backends(jobCounter % backends.size) forward job

    case BackendRegistration if !backends.contains(sender()) =>
      context watch sender()
      backends = backends :+ sender()

    case Terminated(a) =>
      backends = backends.filterNot(_ == a)
      // add code to restart
      if( backends.isEmpty)
         context.system.shutdown
  }
}

object TransformationFrontend {
  
  def main(args: Array[String]): Unit = {
    val port = if (args.isEmpty) "0" else args(0)
    val config = ConfigFactory.parseString(s"akka.remote.netty.tcp.port=$port").
      withFallback(ConfigFactory.parseString("akka.cluster.roles = [frontend]")).
      withFallback(ConfigFactory.load())

  def start: Unit = {
    val system = ActorSystem("ClusterSystem", config)
    system.registerOnTermination{start} 
    
    val frontend = system.actorOf(Props[TransformationFrontend], name = "frontend")

    val counter = new AtomicInteger
    import system.dispatcher
    system.scheduler.schedule(2.seconds, 2.seconds) {
      implicit val timeout = Timeout(5 seconds)
      (frontend ? TransformationJob("hello-" + counter.incrementAndGet())) onSuccess {
        case result => println(result)
      }
    }
   }
   start
  }
}
{% endhighlight %}

The foreground will now reconnect when the backends restart.



