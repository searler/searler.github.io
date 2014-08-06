---
layout: post
title:  "Data Communication Protocol implementation using Scala - Control Characters"
date:   2014-05-04 18:20:00
categories:  scala 
---

The protocol specifies a specific set of control bytes, which are listed below.
The EvenParity apply() serves both to document the required parity and ensure that
parity is appropriately set.

{% highlight scala %}
object Bytes {
  val SOH  = EvenParity(0x01)
  val STX  = EvenParity(0x02)
  val ETX  = EvenParity(0x03)
  val ACK1 = EvenParity(0x06)
  val INV  = EvenParity(0x07)
  val REP  = EvenParity(0x11)
  val RM   = EvenParity(0x12)
  val NAK  = EvenParity(0x15)
  val SYN  = EvenParity(0x16)
  val ETB  = EvenParity(0x17)
  val CAN  = EvenParity(0x18)
  val EM   = EvenParity(0x19)
  val ACK2 = EvenParity(0x1C)
  val WBT  = EvenParity(0x1E)
  val DEL  = EvenParity(127)
  val SEL  = EvenParity('H')
 } 
{% endhighlight %}

It might be argued the above design is too simplistic 
since it does not use any typing mechanism to specify the concept of a control character.
That additional sophistication (and complexity) is not useful for this problem domain.

Some of the control characters are used to control transmission (details to follow).
The PairByte object provides an extractor that detects those special characters.
(The nomenclature arises because these characters appear in pairs)

{% highlight scala %}
object PairByte {
  private val universe = Array(ACK1, ACK2, NAK, RM, WBT, REP, CAN, INV)
  def unapply(byte: Byte) = universe contains byte
}
{% endhighlight %}




