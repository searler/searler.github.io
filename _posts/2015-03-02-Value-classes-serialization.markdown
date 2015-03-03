---
layout: post
title:  "Serialization of Scala value classes"
date:   2015-03-02 20:37:00
categories:  scala kryo
---

Example

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


| Implementation  | Serialization | Kryo |
| --------------  | ------------- | ---- |
| Simple          | 86            | 8    |
| Polymorphic     | 157           | 9    |
| Partial         | 87            | 8    |
| Decorated       | 165           | 9    |













