---
layout: post
title:  "Akka reactive streams subscriber"
date:   2014-04-20 18:00:00
categories: akka scala reactive
---

The  Akka streams [activator](http://www.typesafe.com/activator/template/akka-stream-scala)
provides a few illustrative examples. These are all based on the api, especially the Scala DSL; and
thus do not illustrate all the capabilities. (Quite reasonable given the very early stage of development).

The SPI provides a mechanism to unsubscribe, which lead to a question as to how that might actually be used.
The following code implements a Consumer that prints each line and unsubscribes when "TO" is encountered.

{% highlight scala %}
implicit val system = ActorSystem("Sys")

val text =
    """|Lorem Ipsum is simply dummy text of the printing and typesetting industry.
       |Lorem Ipsum has been the industry's standard dummy text ever since the 1500s, 
       |when an unknown printer took a galley of type and scrambled it to make a type 
       |specimen book.""".stripMargin
val m = FlowMaterializer(MaterializerSettings())

val p = Flow(text.split("\\s").toVector).
        map(line => line.toUpperCase).
        toProducer(m)
      
p.produceTo(new Consumer[String]{
    var subscription:Subscription = null
    def getSubscriber:Subscriber[String] = new Subscriber[String] {
       def onSubscribe(s:Subscription){
           subscription = s;
           s.requestMore(1)}
       def onNext(e:String){
           println(e);
           if(e == "TO")
              subscription.cancel();
           else
              subscription.requestMore(1)}
       def onComplete(){println("done")} //never called
       def onError(t:Throwable){println(t)}
    }
 })
        
 Flow(p).
    onComplete(m) {
       case Success(_) => system.shutdown()
       case Failure(e) =>
          println("Failure: " + e.getMessage)
          system.shutdown()
    }
}
{% endhighlight %}

The Consumer is rather similar to the Reactive Extensions [Observer](https://github.com/Netflix/RxJava/blob/master/rxjava-core/src/main/java/rx/Observer.java). 

