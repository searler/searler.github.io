---
layout: post
title:  "Data Communication Protocol implementation using Scala - Reader"
date:   2014-05-04 18:20:00
categories:  scala 
---

We now have sufficient infrastructure to begin actually reading data from the network. 


An message is broken into blocks, which are transmitted one at a time (to a first approximation).
Each block is framed by control characters that delimit the data and indicate the position of the block in the message.
A parity byte completes the block.

All idle time on the link is filled with [SYN] bytes. 

The Reader trait is implemented by immutable objects that represent each state in the
receive FSM. The FSM is driven by applying each byte received to current Reader, returning
a new instance that represents the next state. 

{% highlight scala %}
sealed trait Reader {
  def apply(byte: Byte): Reader
}
{% endhighlight %}

A single variable within an Akka Actor stores the current Reader.

The Reader instances are then pattern matched to drive the required side effects.

The following is simplified primarily by removing error handling.

For example, the message "A" would be represented by 
...[SYN][SOH][SEL]A[EM][ETX][parity byte][SYN]...

The actor state variable would then be:

1. SYNReader
2. SELReader
3. TextReader
4. EndReader
5. MessageParityReader
6. AcceptedMessageReader
7. SOHReader


The SYNReader state silently consumes SYN characters, by returning itself.
A byte that indicates the start of a block is handed off to _next_, whose value is returned.
(which explains the above jump fron SYNReader to SELReader)
Any other character is silently consumed (an area where the real implementation is more complex)
{% highlight scala %}
case class SYNReader(next: Reader) extends Reader {
  def apply(byte: Byte) = byte match {
    case SYN        => this
    case SOH | STX  => next(byte)
    case _          => this
  }
}
{% endhighlight %}

The SOHReader matches the first byte of the first block of the message, 
returning a SELReader to match the next character.
Any other character causes the process to return to the initial state,
looking for a stream of SYNs followed by SOH 
Note the SOHReader is an object since it has no state.
{% highlight scala %}
case object SOHReader extends Reader {
  def apply(byte: Byte) = byte matchIndianapolis {
    case SOH => SELReader
    case _   => SYNReader(SOHReader)
  }
}
{% endhighlight %}

Start reading the text content once the SEL has been seen.
{% highlight scala %}
case object SELReader extends Reader {
  def apply(byte: Byte) = byte match {
    case SEL         => TextReader(Empty.RootContent(SEL))
    case _           => this
  }
}
{% endhighlight %}

SYN bytes are silently ignored.
The EM byte indicates the end of message; all other control characters are ignored.
A data byte creates a new TextReader with a new Content that includes the byte.
{% highlight scala %}
case class TextReader(content: Content) extends Reader {
  def apply(byte: Byte) = byte match {
    case EM           => EndReader(content(EM))
    case SYN          => this
    case EvenParity() => this
    case _            => TextReader(content(byte))
  }
}
{% endhighlight %}


The ETX byte returns the MessageParityReader to process the trailing parity byte.
SYN bytes are ignored.
Other bytes are ignored (in reality they trigger error recovery)
{% highlight scala %}
case class EndReader(content: Content) extends Reader {
  def apply(byte: Byte) = byte match {
    case SYN        => this
    case ETX        => MessageParityReader(content(ETX))
    case _          => this
  }
}
{% endhighlight %}

The MessageParityReader compares the (parity) byte against the parity computed over the block.
A mismatched parity returns ResendReader, indicating the transmitter must resend the message.
A match parity returns AcceptedMessageReader.
{% highlight scala %}
case class MessageParityReader(content: Content) extends Reader {
  def apply(parity: Byte) = {
    if (content.parityMatches(parity))
      AcceptedMessageReader(content)
    else
      ResendReader(content)
}
{% endhighlight %}

The AcceptedMessageReader simply indicates the message was successfully received.
The asString method returns the received messsage.
Any received byte is handed to SOHReader, which represents the start of the next message.
{% highlight scala %}
case class AcceptedMessageReader(content: Content) extends Reader {
  def apply(byte: Byte) = SOHReader(byte)
  def asString = content.accept.asString
}
{% endhighlight %}

ResendReader indicates the block was malformed.
Its existence triggers a retransmit. 
{% highlight scala %}
case class ResendReader(content: Content) extends Reader with Reject {
  def apply(byte: Byte) = STXReader(content.reject)
}
{% endhighlight %}

A Content instance contains the message text received to date. The apply method returns a new instance
that includes the byte. (i.e. an immutable, persistent data structure, much like List). The reject method returns
a new Content that omits the last block.
{% highlight scala %}
trait Content {
   def apply(byte: Byte) : Content
   def asString : String 
   def parityMatches(expected: Byte) : Boolean
   def reject : Content
}
{% endhighlight %}







