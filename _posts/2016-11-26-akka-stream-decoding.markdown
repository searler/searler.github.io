---
layout: post
title:  "Framing and decoding with Akka Streams"
date:   2016-11-26 12:31:00
categories:  scala akka
---

The use case is:

1. Connect to a TCP server that delivers a data stream (from a Terminal Server in the real world)
2. Extract a message that is framed by a SOM and length field
3. Decode the message into its length and payload (real world has many fields)

This code is most conveniently tested via netcat, using human readable hex input.

{% highlight scala %}
val tcpFlow = Tcp().outgoingConnection(new InetSocketAddress("127.0.0.1", 2558))
val bytesFlow = tcpFlow.map(bs => ByteVector.fromValidHex(bs.utf8String).toByteString)
val bitsFlow = bytesFlow.statefulMapConcat[BitVector](
   () => new SOMEscapedFramer(BitVector.apply))
val decodedFlow = bitsFlow.map(bits => codec.decode(bits).require.value)
decodedFlow.runWith(Source.maybe, Sink.foreach(println))
{% endhighlight %}

* tcpFlow connects to server at 127.0.0.1 port 2558
* bytesFlow converts ByteString to hex string representation to ByteString.
Only exists to support test via netcat.
* bitsFlow frames ByteStrings into a stream of zero or BitVectors. 
Each BitVector contains a fully framed message or "noise" bytes that appear between the frames.
* decodedFlow is a stream of (length,payload)
* runWith executes the flow, printing the decoded tuples.


Use [scodec](http://scodec.org/) to extract the length field and following bytes.
{% highlight scala %}
val codec = uint8 ~ bytes
{% endhighlight %}

Extract a frame that starts with a SOM byte (0x2b), followed by a one byte unsigned length field. 
An escape byte (0x10) is used when SOM or escape appears within the payload.

The instance is stateful since a valid frame can span many incoming ByteStrings, requiring the buffering 
of the data received to this point. A single ByteString might also contain several frames.

The bytes outside of the frame are also delivered, to support error reporting.
The real code has checksums and precise decoding of the structure so the two cases are easily distinguished.
{% highlight scala %}
class SOMEscapedFramer[T](create:ByteBuffer => T) extends 
   Function1[ByteString, immutable.Iterable[T]] {

    val escape: Byte = 0x10
    val som: Byte = 0x2b

    sealed trait State

    case object AwaitStart extends State

    case object AwaitLength extends State

    case object Gathering extends State

    val buff = ByteBuffer.allocate(1024)

    var state: State = AwaitStart
    var length = 0
    var seenEscape = false

    def apply(bytes: ByteString): Iterable[T] = {
      var result = List.empty[T]

      def complete = {
        buff.flip
        result = result :+ create(buff)
        buff.clear
      }

      def add(b: Byte) = {
        buff.put(b)
        state match {
          case AwaitLength =>
            length = b + 3
            state = Gathering
          case Gathering =>
            if (buff.position == length) {
              complete
              state = AwaitStart
            }
          case _ =>
        }
      }


      for (b <- bytes) {
        if (seenEscape) {
          seenEscape = false
          add(b)
        } else {
          b match {
            case `som` =>
              state = AwaitLength
              if (buff.position > 0)
                complete
            case `escape` =>
              seenEscape = true
            case _ =>
              add(b)
          }
        }
      }
      result
    }
  }
{% endhighlight %}








