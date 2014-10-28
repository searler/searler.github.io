---
layout: post
title:  "Replacing phantom types with implicit evidence"
date:   2014-10-27 20:20:00
categories:  scala 
---

The [phantom types]({% post_url 2014-10-26-phantom_door %}) can be replaced
by implicit evidence as follows

{% highlight scala %}
class Door[State <: DoorState] {
  def open(implicit ev: State =:= Closed): Door[Open] = this.asInstanceOf[Door[Open]]
  def close(implicit ev: State =:= Open): Door[Closed] = this.asInstanceOf[Door[Closed]]
}
  
class Room[State <: DoorState](val door: Door[State]) {
  def closeDoor(implicit ev: State =:= Open) = this.asInstanceOf[Room[Open]]
  def openDoor(implicit ev: State =:= Closed) = this.asInstanceOf[Room[Closed]]
}
{% endhighlight %}

The style might be considered a little more explicit and perhaps easier to understand.
Note that there is additional overhead since instances of the evidence class will be
created and passed as arguments. 

The fundamental disadvantages remain unchanged. 

This [blog post](http://covariantblabbering.blogspot.com/2014/02/building-with-phantom-types.html) describes
a more powerful form of the phantom type implementation. 

Most phantom type examples are [builders](http://en.wikipedia.org/wiki/Builder_pattern), which might perhaps be their only niche. 








