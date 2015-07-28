---
layout: post
title:  "Scala curiosity using multiple inheritance"
date:   2015-7-27 18:20:00
categories:  scala 
---

A new project requires a large number of scodec Codecs and the associated domain model.
A sealed trait with derived class objects and classes is a good match for much of the problem.

A simple transcription of the ICD lead to the following code:

{% highlight scala %}
sealed trait Warning
case object Closed extends Warning
case object Warn extends Warning
  
sealed trait Failure
case object Closed extends Failure
case object Failed extends Failure
{% endhighlight %}
Which is of course illegal since Closed appears twice. 
(The terminology is weird but matches the source documentation)

Making Closed implement both Traits works fine, although it does look a little strange.

{% highlight scala %}
sealed trait Warning
case object Warn extends Warning
  
sealed trait Failure
case object Failed extends Failure
  
case object Closed extends Failure with Warning
{% endhighlight %}

Arguably Failure and Warning are now coupled to some extent, although I cannot see any adverse consequences.
In any event, that would be irrelevant in this context.









