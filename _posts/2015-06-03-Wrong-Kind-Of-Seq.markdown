---
layout: post
title:  "Wrong kind of Seq"
date:   2015-06-03 20:07:00
categories:  scala Akka reactive streams
---

The API contains the following factory method

{% highlight scala %}
def apply[T](iterable: Iterable[T]): Source[T, Unit]
{% endhighlight %}

With an example
{% highlight scala %}
Source(Seq(1,2,3))
{% endhighlight %}

This fails to compile with a huge error message referencing an overload mismatch.

This failure occurs since the default Seq is mutable and the Akka API expects the immutable form.
It is necessary to explicitly import the immutable form.








