---
layout: post
title:  "The Fletcher checksum and scodec"
date:   2014-09-06 17:00:00
categories:  scala scodec
---

The [Fletcher checksum](http://en.wikipedia.org/wiki/Fletcher's_checksum) could be computed using the 
[scodec](https://github.com/scodec/scodec) infrastructure as follows:

{% highlight scala %}
object FletcherChecksum {

  def apply(bv:BitVector): BitVector = {
    val csum = bv.toByteArray.foldLeft((0, 0)) { (p, b) =>
        val lsb = (p._2 + (0xff & b)) % 255
        ((p._1 + lsb) % 255, lsb)
    }
    bv ++BitVector(csum._1.asInstanceOf[Byte], csum._2.asInstanceOf[Byte])
  }
}
{% endhighlight %}
Note the checksum is appended to the input BitVector.

With a test
{% highlight scala %}
@Test
def wikipedia {
   assertEquals(BitVector(0x01, 0x02, 0x04, 0x03), 
      FletcherChecksum(BitVector(0x01, 0x02)))
}
{% endhighlight %}

This design is not consistent with the [scodec cryptographic signing functionality](http://scodec.github.io/scodec/latest/api/index.html#scodec.codecs.package).
Unfortunately that code is built around the Java security API and the core 
```java.security.Signature``` class is not appropriate for very much simpler checksum functionality.







