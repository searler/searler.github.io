---
layout: post
title:  "Serialization of Scala value classes"
date:   2015-03-02 20:37:00
categories:  scala kryo
---

Using primitives can be error prone and hard to understand in large argument lists, since
it difficult to determine the identity of a value. Named parameters are helpful but
really all values should use a domain type.

Scala provides value classes to implement domain types without incurring the
associated runtime costs. In particular, it avoids the need to allocate an instance
on the heap. 

The documentation (AFAIK) does not cover the serialization costs. 

The following code defines a simple domain concept, using four different designs.

Samples are then serialized, using Java Serialization and [Kryo](https://github.com/EsotericSoftware/kryo);
with the number of bytes listed in the table below.

The numbers indicate that value classes have no impact.

{% highlight scala %}
 case class Simple(id: Int, form: Int, odd: Boolean, value: Float)

  trait Data
  case class Odd(v: Float) extends Data
  case class Even(v: Float) extends Data

  case class Polymorphic(id: Int, form: Int, value: Data)

  case class Id(n: Int) extends AnyVal
  case class Type(t: Int) extends AnyVal

  trait DataValue extends Any
  case class OddValue(v: Float) extends AnyVal with DataValue
  case class EvenValue(v: Float) extends AnyVal with DataValue

  case class Decorated(id: Id, form: Type, value: DataValue)

  case class Kind(b: Boolean) extends AnyVal
  case class Value(f: Float) extends AnyVal

  case class Partial(id: Id, kind: Type, odd: Kind, value: Value)
{% endhighlight %}


| Implementation  | Java | Kryo |
| --------------  | ------------- | ---- |
| Simple          | 86            | 8    |
| Polymorphic     | 157           | 9    |
| Partial         | 87            | 8    |
| Decorated       | 165           | 9    |













