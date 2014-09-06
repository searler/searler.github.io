---
layout: post
title:  "The crippled codec"
date:   2014-09-06 16:20:00
categories:  scala 
---

The [scodec](https://github.com/scodec/scodec) library implements a combinator based encoding and decoding library for Scala.

My use case only required encoding, but all the combinators are designed for codecs (which are bidirectional).
I could write an additional set of encoder only combinators, but that seemed rather pointless. 
Rather I created a simple mechanism to "stub out" the unused decoder functionality. 

A primary reason for ignoring the decoder path was to maximize the _win_ over a legacy Java implementation! 
The LOSC reduction was little disappointing, being only 15x. That is not much better than the 10x average seen
for other legacy Java re-implementations

Consider the implementation of a two digit Binary Coded Decimal number

{% highlight scala %}
val bcd2: Codec[Int] = (uint4 ~ uint4).xmap(
     v => v._1 * 10 + v._2,
     d => (d / 10) ~ (d % 10))
{% endhighlight %}

This could be reduced to
{% highlight scala %}
val bcd2: Codec[Int] = (uint4 ~ uint4).xmap(
     null,
     d => (d / 10) ~ (d % 10))
{% endhighlight %}
which will generate a NullPointerException if the decoder path is referenced.
This design would probably result in howls of derision from the Scala community.

{% highlight scala %}
val bcd2: Codec[Int] = (uint4 ~ uint4).xmap(
     _ => ???,
     d => (d / 10) ~ (d % 10))
{% endhighlight %}
Would be a little more idiomatic.
It will still fail if the decoder path is called but that cannot be avoided.

This design clutters the code and still leaves the decoder path visible (and subject to error).
What we need is a combinator that only provides the encoding path.

It is unlikely that many users of the scodec library would find this combinator useful, so it cannot
be sensibly added to the existing combinators. The pimp-my-library pattern via an implicit class would be 
more appropriate.

{% highlight scala %}
private implicit class CrippledCodec[A](c: Codec[A]) {
    def apply[B](g: (B) => A): Codec[B] = c.xmap(_ => ???, g)
}
{% endhighlight %}

With the final code being
{% highlight scala %}
val bcd2: Codec[Int] = (uint4 ~ uint4)(d => (d / 10) ~ (d % 10))
{% endhighlight %}


Note that this formulation does not compile, complaining that ```toInt``` is not a member of ```B```
{% highlight scala %}
private implicit class CrippledCodec[A,B](c: Codec[A]) {
    def apply(g: (B) => A): Codec[B] = c.xmap(_ => ???, g)
}
{% endhighlight %}




