---
layout: post
title:  "Woes generating Json with Scala "
date:   2015-08-15 17:20:00
categories:  scala json  
---

A recent project needed a web interface to display Scala case classes populated by [scodec](http://scodec.org/).
The UI is simply a developer level dump of the data, so there are no specific constraints placed on the Json needed
to populate the pages.

The domain model is fairly complex  and needs to support the non trivial codecs.
The design makes extensive use of package objects, traits and case objects. 
The last two require a type indicator in the Json.

Our Java projects made extensive use of [xstream](http://x-stream.github.io/) to
implement similar non-Java specific serialization. Xstream "just works", at the cost of using
reflection. 

 The first attempt used [upickle](https://github.com/lihaoyi/upickle-pprint). That was initially successful, and
 correctly handled the traits and the objects. However, I quickly ran into a known [bug](https://github.com/lihaoyi/upickle-pprint/issues/68)
 which originates from  a long standing  [scalac  bug](https://issues.scala-lang.org/browse/SI-7046)
 
The next attempt tried [json4s](https://github.com/json4s/json4s) (nee Lift Json).
That code base is fairly old (and evidently no longer actively developed). Some custom code was required to
handle the package object and case object class names.


{% highlight scala %}
import org.json4s._

/**
* Types from package object contain package$
*
* Object has $ suffix
*
 * Assume class name does not contain $
*/
case class PackageTypeHints(hints: List[Class[_]]) extends TypeHints {
  def hintFor(clazz: Class[_]) = {
    var name = clazz.getName
    if (name.endsWith("$"))
      name = name.substring(0, name.length - 1)
    name.substring(name.lastIndexWhere(c => c == '.' || c == '$') + 1)
  }
  def classFor(hint: String) = hints find (hintFor(_) == hint)
}

{% endhighlight %}

This attempt also failed when the compiler was unable to resolve all the implicits. 

The final attempt applied the [Play](https://www.playframework.com/) Json functionality. That did require manual effort
to define the Writers, but was ultimately successful. That implementation deserves its own post.

The above experience unfortunately implies that the "code generation" tools of Scala (macros, implicits, etc) exceed the current
capabilities of scalac for non trivial codebases.







