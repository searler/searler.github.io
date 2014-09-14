---
layout: post
title:  "Padding in scodec"
date:   2014-09-14 12:00:00
categories: scodec scala 
---

The following implements a simple codec using [scodec](https://github.com/scodec/scodec)

{% highlight scala %}
case class Data(a:Int, b:Int)
case class Container(d:IndexedSeq[Data])
 
val dataCodec = (uint4 ~ uint4).pxmap(Data.apply,Data.unapply)
  
val containerCodec:Codec[Container] = (repeated(dataCodec)).xmap(
   Container.apply,
   c => c.d
)
{% endhighlight %}

The result is a representation whose size depends on number of Data instances in the Container.
Many use cases require a fixed size representation.
The following code pads to 5 Data instances, i.e. 40 bits.
This approach is only acceptable if the use case requires trailing padding with zero bits.
Note also the unfortunate coupling to the size of Data representation.

{% highlight scala %}
val containerCodec:Codec[Container] = fixedSizeBits(5*8,repeated(dataCodec)).xmap(
    Container.apply,
    c => c.d
) 
{% endhighlight %}


This code pads the representation with Data(13,14).
The two limitations of fixedSizeRepresentation no longer apply and the coupling
to the size of the Data representation is eliminated.
However, it does require that the pad value can be represented by Data. 
The pad values are often illegal within the domain and would be rejected by any validation attached to Data.
{% highlight scala %}
val containerCodec:Codec[Container] = (repeated(dataCodec) ~ repeated(dataCodec)).xmap(
   p => Container(p._1),
   c => c.d ~ Vector().padTo(5 - c.d.length, Data(13,14))
)
{% endhighlight %}

This code pads the representation with specific bytes. 
It allows the usage of any pad values, at the cost of even greater coupling to
the Data representation.
{% highlight scala %}
val containerCodec:Codec[Container] = (repeated(dataCodec) ~ repeated(constantLenient(0xd,0xe))).xmap(
  p => Container(p._1),
  c => c.d ~ Vector().padTo(5 - c.d.length, ())
)
{% endhighlight %}

This code resolves all of the above issues, by providing a separate codec to
generate the pad representation.

There is some coupling between the tuples that represent the pad value and Data.
That seems unavoidable and the compiler will help to catch many of the possible
errors.
{% highlight scala %}
val rootCodec = (uint4 ~ uint4)
 
val dataCodec = rootCodec.pxmap(Data.apply,Data.unapply)
  
val padCodec:Codec[Unit]= rootCodec.xmap(
  _ => (),
  _ => 13 ~ 14
)
  
val containerCodec:Codec[Container] = (repeated(dataCodec) ~ repeated(padCodec)).xmap(
  p => Container(p._1),
  c => c.d ~ Vector().padTo(5 - c.d.length, ())
)
{% endhighlight %}

The above code is suboptimal if the cost of encoding the pad value is high.
The following caches the result of the encoding.
{% highlight scala %}
val paddedCodec = {
     val padCodec:Codec[Unit]= rootCodec.xmap(
         _ => (),
         _ => 13 ~ 14)
     constantLenient(padCodec.encodeValid(()))
}
  
val containerCodec:Codec[Container] = (repeated(dataCodec) ~ repeated(paddedCodec)).xmap(
   p => Container(p._1),
   c => c.d ~ Vector().padTo(5 - c.d.length, ())
)
{% endhighlight %}

Finally an example where the padding leads the actual data.
{% highlight scala %}  
val containerCodec:Codec[Container] = (repeated(paddedCodec) ~ repeated(dataCodec)).xmap(
   p => Container(p._2),
   c => Vector().padTo(5 - c.d.length, ())  ~ c.d
)
{% endhighlight %}

It might be reasonable to extend the standard scodec combinators to better handle these cases.




