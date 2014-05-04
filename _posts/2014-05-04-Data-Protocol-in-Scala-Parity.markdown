---
layout: post
title:  "Data Communication Protocol implementation using Scala - Parity"
date:   2014-05-04 18:10:00
categories:  scala 
---

The protocol uses parity to distinguish between control (even) and data (odd) characters.
Obviously data is then limited to 7 bit ASCII.

The Parity trait provides 

* apply(b:Byte):Byte return b with 8th (parity) bit set to achieve the specified parity.
* unapply(b:Byte):Boolean returns true iff b has the required parity.

The EvenParity and OddParity objects then implement the trait, providing the appropriate value for the abstract value _remainder_

{% highlight scala %}
sealed trait Parity {
  protected val remainder: Int
  private def conforms(b: Byte) = Integer.bitCount(b) % 2 == remainder
  def apply(b: Byte) = if (conforms(b)) b else ((b | 0x80).asInstanceOf[Byte])
  def unapply(b: Byte) = conforms(b)
}

object EvenParity extends Parity {
  val remainder = 0
}

object OddParity extends Parity {
  val remainder = 1
}
{% endhighlight %}


The protocol requires a parity byte, computed over a block of bytes.
That block is represented by a Seq, so as to provide maximum flexibility.
{% highlight scala %}
def parity(bytes:Seq[Byte]) = bytes.foldLeft(0) { (p, b) => p ^ b }.asInstanceOf[Byte]
{% endhighlight %}




