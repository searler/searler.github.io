---
layout: post
title:  "Composing scodec-bit hex representation"
date:   2015-02-16 16:20:00
categories:  scala scodec
---

The [scodec-bits](https://github.com/scodec/scodec-bits) project provides convenient mechanisms to specify constant data:

{% highlight scala %}
val x: ByteVector = hex"deadbeef"
val y: BitVector = bin"00101101010010101"
{% endhighlight %}

This syntax is quite clear until you need to assemble a ByteVector from pieces:

{% highlight scala %}
val prefixed: ByteVector = hex"0102$x"
{% endhighlight %}

Note that the referenced variable must be a ByteVector!

Placing the substitution within the string requires an expression, rather
than the simple variable reference. 

{% highlight scala %}
val wrapped: ByteVector = hex"0102${x}0304"
{% endhighlight %}

An alternative is to explicitly concatenate the ByteVectors:

{% highlight scala %}
val prefixed: ByteVector = hex"0102" ++ x
{% endhighlight %}











