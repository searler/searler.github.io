---
layout: post
title:  "Reconsidering the cake pattern"
date:   2014-10-27 21:-0:00
categories:  scala 
---

This set of [best practices](https://github.com/alexandru/scala-best-practices) contains some interesting
ideas. 

The [recommendation](https://github.com/alexandru/scala-best-practices/blob/master/sections/3-architecture.md#31-should-not-use-the-cake-pattern) 
to avoid the [Cake Pattern](http://lampwww.epfl.ch/~odersky/papers/ScalableComponent.pdf), combined with a [DI suggestion](https://github.com/alexandru/scala-best-practices/blob/master/sections/2-language-rules.md#24-should-not-define-useless-traits)
are particularly interesting.

The Cake Pattern is essentially an OO pattern that uses _Inheritance for Implementation_, which is generally considered an anti-pattern.
OO design standards have evolved with experience, recommending composition over inheritance. 


The conventional Java interface based DI 
{% highlight scala %}
trait DBService {
  def getAssets: Future[Seq[(AssetConfig, AssetPersistedState)]]
}
{% endhighlight %}
is replaced by a flat, functional design
{% highlight scala %}
f: => Future[Seq[(AssetConfig, AssetPersistedState)]]
{% endhighlight %}










