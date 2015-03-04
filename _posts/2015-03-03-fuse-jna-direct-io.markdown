---
layout: post
title:  "Streaming data via fuse-jna"
date:   2015-03-03 18:20:00
categories:  Fuse jna 
---

The [fuse-jna project](https://github.com/EtiennePerot/fuse-jna) provides an convenient mechanism to
implement a [user space file system](http://fuse.sourceforge.net/), using Java.

I encountered a need to stream data using custom serial hardware that is accessed via
a proprietary Java API. With fuse-jna, the API can be mapped into a file system that contains
a file for each port. 

The only difficulty was reading the data from the port. The [read](http://fuse.sourceforge.net/doxygen/structfuse__operations.html#a2a1c6b4ce1845de56863f8b7939501b5)
expects the implementation to fill the specified buffer. fuse appears to assume that a half filled buffer indicates EOF.

This problem was resolved by using direct-io, which is easily enabled in the open implementation.

{% highlight java %}
public int open(String path, FileInfoWrapper info) {
      info.direct_io(true);
       ...
}       
{% endhighlight %}








