---
layout: post
title:  "flow library raises API question"
date:   2014-06-01 16:00:00
categories: akka scala reactive
---

The [flow](https://github.com/jodersky/flow) is an interesting serial port library

* Not RxTx (whose replacement is long overdue)
* Uses Akka IO

The native code infrastructure is also interesting and might be useful in other projects.
That will require additional study. 

The API has a surprising wrinkle, that might be worthy of further discussion.

[Serial.scala](https://github.com/jodersky/flow/blob/master/flow/src/main/scala/com/github/jodersky/flow/Serial.scala)
has a method
{% highlight scala %}
  case class Write(data: ByteString, ack: Int => Event = NoAck) extends Command

  case object NoAck extends Function1[Int, Event] {
    def apply(length: Int) = sys.error("cannot apply NoAck")
  }
{% endhighlight %}

The sys.error means `NoAck(12)` would throw a RuntimeException.
That obviously does not occur and further investigation turns up this implementation
{% highlight scala %}
if (ack != NoAck) sender ! ack(sent)
{% endhighlight %}

This is essentially a null pointer check, which would not generally be considered Idiomatic Scala.
After some experience with Scala, writing `if` is starting to feel as inappropriate as writing `goto`.

NoAck is also an example of the "uninherit" anti-pattern, where the derived implementation attempts to ignore the code provided by the base class. 
The Java equivalent would throw `UnsupportedOperationException`.

The canonical Scala (and Java 8) solution would use Option
{% highlight scala %}
case class Write(data: ByteString, ack: Option[Int => Event] = None) extends Command
{% endhighlight %}
or
{% highlight scala %}
case class Write(data: ByteString, ack: Int => Option[Event] = NoAck) extends Command

case object NoAck extends Function1[Int, Option[Event]] {
   def apply(length: Int) = None
}
{% endhighlight %}
The if test might have better performance, but that is unlikely to be a concern for code that drives a serial port.

Consider the usage of `ack: Int => Event`.
The sample [Terminal.scala](https://github.com/jodersky/flow/blob/master/flow-samples/terminal/src/main/scala/com/github/jodersky/flow/samples/terminal/Terminal.scala) has this usage
{% highlight scala %}
case class Wrote(data: ByteString) extends Event
...
case Wrote(data) => log.info(s"Wrote data: ${formatData(data)}")
...
val data = ByteString(input.getBytes)
operator ! Write(data, length => Wrote(data.take(length)))
{% endhighlight %}
Which means `data` is referenced in the `receive` method of the `operator` actor. 
In this case, there is no problem because `data` is immutable.

Note that `ack` is only useful if it closes over state, which will require case to ensure well defined behavior.




 




