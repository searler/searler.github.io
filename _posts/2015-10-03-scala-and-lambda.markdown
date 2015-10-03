---
layout: post
title:  "Mixing Scala and Java lambdas"
date:   2015-10-03 16:50:00
categories:  scala 
---

The following code is extracted from a Scala project that parses a stream of marshalled binary
messages. The code that actually unmarshals the bytes is defined by ```Unmarshaller```.

{% highlight scala %}
type Unmarshaller = (Int , Adapter)=>Payload

private def payload(unmarshal:Unmarshaller)(id: Int) = new Decoder[Payload] {
   def decode(bits: BitVector): Attempt[DecodeResult[Payload]] = {
     val t = Try(Attempt.successful(DecodeResult(
       unmarshal(id,
         new AdapterImplementation(bits.toByteBuffer)),
       BitVector.empty)))
     t.getOrElse(Attempt.failure(InsufficientBits(8, 0, Nil)))
   }
}
{% endhighlight %}

The Scala code implements a test harness for a Java system. That Java (only) codebase also has to 
unmarshal the byte stream and would thus be convenient to make the Unmarshaller code common 
between the two codebases.

The Java 8 lambdas provide an equivalent representation.

{% highlight scala %}
type Unmarshaller = BiFunction[Integer,Adapter,Payload]
{% endhighlight %}

Note that is the _only_ change required in the Scala code. The scala compiler desugars
```unmarshal(x,y)``` to ```unmarshal.apply(x,y)```, which corresponds to the ```BiFunction``` API.

Unfortunately, the Java API does not consistently use ```apply```, so this syntactic convenience
is limited. For example ```(Int)=>Unit``` corresponds to ```Consumer<Unit>```, whose API 
defines an ```accept``` method.
