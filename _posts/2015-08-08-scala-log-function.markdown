---
layout: post
title:  "Using functional interfaces for logging DI"
date:   2015-08-08 15:20:00
categories:  scala 
---

A very simple example that illustrates the benefits of the functional over a more familiar OO approaches.

The functionality of the WrappingCodec (below) requires the logging of invalid data. 
The runtime environment is provided by Akka, so use of their [LoggingAdapter](http://doc.akka.io/api/akka/2.3.12/#akka.event.LoggingAdapter) 
would be simple. However, this code has no other Akka dependencies and it would be unfortunate to mandate
a coupling to a large framework.

The standard OO design would require an interface, that the WrappingCodec calls, with an implementation
that calls the LoggingAdapter. Two levels of adaptation is undesirable, even if it would not be out
of place in many Java enterprise codebases. 

The alternative is to simply specify that the WrappingCodec takes a function parameter, of 
type ```String=>Unit```. That specifies the minimal requirement, allowing the most general implementation.

{% highlight scala %}
class WrappingCodecs[A, B, R](
    val decoder: Decoder[A],
    val encoder: Encoder[B],
    val convert: Codec[R],
    val f: A => B,
    log: String => Unit)
{% endhighlight %}

The actual usage does not require any additional code since the LoggingAdapter already conforms.

{% highlight scala %}
val log:LoggingAdapter= ...

val codecs = new WrappingCodecs(CMSCodecs.request,
        CMSCodecs.response,
        ByteStringCodec,
        manager.apply,
        log.error)
{% endhighlight %}

This approach is somewhat limited since it does not provide any means to provide additional
metadata, such as level and context. Those were not required in this particular application.








