---
layout: post
title:  "Are Scala phantom types really useful?"
date:   2014-10-26 10:20:00
categories:  scala 
---

[This](http://www.slideshare.net/ktoso/scala-types-of-types-lambda-days)

{% highlight scala %}
sealed trait DoorState
final class Open extends DoorState
final class Closed extends DoorState

class Door[State <: DoorState] {
  def open[T >: State <: Closed](): Door[Open] = this.asInstanceOf[Door[Open]]
  def close[T >: State <: Open](): Door[Closed] = this.asInstanceOf[Door[Closed]]
}

object Door {
   def create() = new Door[Closed]()
}
{% endhighlight %}

The above code bahave as promised, rejecting at compile time any attempt to open a door
that is already open. 

The semantics are rather strange, since the Door is actually stateless.
Only the compiler knows whether the door is open or closed!

Note how the open and closed Door are the exact same instance.
{% highlight scala %}
val dcls = Door.create  
val dopn = dcls.open  
 
dcls eq dopn  // true
cls == dopn   // true
{% endhighlight %}

The type of the values is rather strange.
The repl reports the expected type for ```dopn``` and ```dopn.close``` functions as expected.
{% highlight scala %}
val dcls:Door[Closed] = new Door[Closed]()
val dopn: Door[Open] = dcls.open // fails to compile
{% endhighlight %}

This mechanism appears to be overly clever, with limited real world applicability


Consider a more complex domain model, with a Room that contains a single door.
{% highlight scala %}
class Room[State <: DoorState](val door:Door[State]){
  def closeDoor[T >: State <: Open] = this.asInstanceOf[Room[Open]]
  def openDoor[T >: State <: Closed] = this.asInstanceOf[Room[Closed]]
} 
{% endhighlight %}

Note how the details of the ```Door``` implementation appear within ```Room```.
In fact, there is no reason to even reference the actual ```Door``` class. 

Now consider scaling this model to cover multiple Doors per Room, Rooms in a Wing of a Building in a Campus.







