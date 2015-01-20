---
layout: post
title:  "Simple Pattern Matching Command Parser"
date:   2015-01-14 08:20:00
categories:  scala
---
[This](https://github.com/RBMHTechnology/eventuate/blob/master/src/test/scala/com/rbmhtechnology/example/OrderExample.scala) code
is a nice illustration of how Scala Pattern Matching simplifies common tasks.

It simply splits the line on spaces and then matches the resultant list.

The code is a little simplistic. For example, the parameters are limited to Strings.

{% highlight scala %}
case line: String => line.split(' ').toList match {
      case "state"                 :: Nil => manager ! GetState
      case "count"   :: id         :: Nil => view    ! GetUpdateCount(id)
      case "create"  :: id         :: Nil => manager ! CreateOrder(id)
      case "cancel"  :: id         :: Nil => manager ! CancelOrder(id)
      case "add"     :: id :: item :: Nil => manager ! AddOrderItem(id, item)
      case "remove"  :: id :: item :: Nil => manager ! RemoveOrderItem(id, item)
      case "resolve" :: id :: idx  :: Nil => manager ! Resolve(id, idx.toInt)
      case       Nil => prompt()
      case "" :: Nil => prompt()
      case na :: nas => println(s"unknown command: ${na}"); prompt()
}
{% endhighlight %}





